---
icon: windows
---

# Introducción y Conceptos Básicos

Active Directory (AD) es el servicio de directorio de Microsoft que centraliza la **autenticación y autorización** en entornos Windows corporativos. Gestiona usuarios, equipos, grupos, políticas y recursos de una red.

Desde el punto de vista ofensivo, comprometer AD equivale a comprometer **toda la organización**: acceso a todos los equipos, datos, servicios y sistemas del dominio.

### Componentes clave

#### Dominio

Unidad administrativa principal. Todos los objetos (usuarios, equipos, grupos) pertenecen a un dominio.

```
ejemplo.local
corp.empresa.com
empresa.internal
```

#### Domain Controller (DC)

El servidor que aloja y gestiona AD. Es el objetivo principal en cualquier ataque. Ejecuta servicios críticos:

* **Kerberos** (puerto 88) — autenticación
* **LDAP** (puerto 389/636) — consultas al directorio
* **SMB** (puerto 445) — compartición de archivos y recursos
* **DNS** (puerto 53) — resolución de nombres del dominio
* **RPC** (puerto 135) — comunicación entre servicios

#### Forest y Trust

```
Forest: conjunto de dominios que comparten el mismo esquema
Tree: dominios con namespace jerárquico (empresa.com, filial.empresa.com)
Trust: relación que permite a usuarios de un dominio acceder a recursos de otro

Tipos de trust:
- One-way:   DomA confía en DomB (usuarios de B acceden a A, no al revés)
- Two-way:   confianza mutua (frecuente entre dominios del mismo forest)
- Transitive: la confianza se propaga (A confía en B, B confía en C → A confía en C)
```

#### Organizational Units (OUs)

Contenedores para organizar objetos del dominio. Se usan para aplicar GPOs y delegar administración.

```
OU=IT,DC=empresa,DC=local
OU=Usuarios,DC=empresa,DC=local
OU=Servidores,DC=empresa,DC=local
```

### Objetos principales de AD

| Objeto       | Descripción                       | Atributo clave                        |
| ------------ | --------------------------------- | ------------------------------------- |
| **User**     | Cuenta de usuario                 | `sAMAccountName`, `userPrincipalName` |
| **Computer** | Cuenta de equipo (termina en `$`) | `dNSHostName`                         |
| **Group**    | Agrupación de objetos             | `member`, `memberOf`                  |
| **GPO**      | Directiva de grupo                | `gPCFileSysPath`                      |
| **OU**       | Contenedor organizativo           | `distinguishedName`                   |
| **SPN**      | Service Principal Name            | Vincula servicio a cuenta             |

### Autenticación en Active Directory

#### NTLM

```
1. Cliente envía el nombre de usuario al servidor
2. Servidor responde con un challenge (número aleatorio de 8 bytes)
3. Cliente cifra el challenge con el hash NTLM de la contraseña
4. Servidor verifica contra el hash almacenado en el DC

Hash NTLM = MD4(UTF-16LE(password))
Formato en SAM/NTDS: usuario:RID:LM_hash:NT_hash
```

#### Kerberos (protocolo principal en AD moderno)

```
AS-REQ  →  Cliente pide TGT al DC (Authentication Server)
AS-REP  ←  DC devuelve TGT cifrado con krbtgt hash
TGS-REQ →  Cliente pide Service Ticket presentando TGT
TGS-REP ←  DC devuelve Service Ticket cifrado con hash de la cuenta de servicio
AP-REQ  →  Cliente presenta Service Ticket al servicio
```

**Tickets:**

* **TGT (Ticket Granting Ticket)**: prueba de identidad, cifrado con el hash de `krbtgt`
* **TGS / Service Ticket**: acceso a un servicio específico, cifrado con el hash de la cuenta de servicio
* **PAC (Privilege Attribute Certificate)**: incluido en los tickets, contiene los grupos del usuario

### Conceptos de seguridad esenciales

#### SID (Security Identifier)

Identificador único de cada objeto en AD.

```
S-1-5-21-[domainID]-[RID]

RIDs conocidos:
500 → Administrator (cuenta local admin predefinida)
502 → krbtgt (cuenta del servicio Kerberos)
512 → Domain Admins
513 → Domain Users
516 → Domain Controllers
519 → Enterprise Admins
520 → Group Policy Creator Owners
```

#### ACL / ACE (Access Control Lists / Entries)

Definen qué objetos tienen qué permisos sobre otros objetos de AD.

```
Permisos peligrosos (abusables para escalada):
- GenericAll      → control total sobre el objeto
- GenericWrite    → modificar atributos del objeto
- WriteOwner      → cambiar el propietario
- WriteDACL       → modificar la ACL del objeto
- ForceChangePassword → resetear contraseña sin saber la actual
- AllExtendedRights → todos los derechos extendidos
- AddMember       → añadir miembros a grupos
- Self            → asignarse derechos a uno mismo
```

#### Grupos privilegiados

| Grupo                       | Privilegios                                             |
| --------------------------- | ------------------------------------------------------- |
| **Domain Admins**           | Admin completo del dominio                              |
| **Enterprise Admins**       | Admin de todo el forest                                 |
| **Schema Admins**           | Modificar el esquema de AD                              |
| **Backup Operators**        | Hacer backup de cualquier archivo (incluyendo NTDS.dit) |
| **Account Operators**       | Crear/modificar cuentas sin ser DA                      |
| **Server Operators**        | Administrar DCs                                         |
| **Remote Management Users** | Acceso WinRM remoto                                     |
| **DNSAdmins**               | Potencial escalada a DA via DLL injection               |

#### GPO (Group Policy Object)

Directivas aplicadas a OUs para configurar equipos y usuarios. Si un atacante puede modificar una GPO aplicada a un DC → Domain Admin.

#### SYSVOL

Carpeta compartida en todos los DCs que contiene scripts de login y GPOs. Accesible por todos los usuarios del dominio.

```
\\dominio\SYSVOL\dominio\Policies\
\\dominio\SYSVOL\dominio\scripts\
```

### Puertos y servicios clave

| Puerto    | Protocolo/Servicio    | Uso en AD                        |
| --------- | --------------------- | -------------------------------- |
| 53        | DNS                   | Resolución de nombres            |
| 88        | Kerberos              | Autenticación                    |
| 135       | RPC                   | Comunicación entre servicios     |
| 139/445   | SMB/NetBIOS           | Compartición, movimiento lateral |
| 389/636   | LDAP/LDAPS            | Consultas al directorio          |
| 464       | Kerberos (cambio pwd) | Cambio de contraseñas            |
| 3268/3269 | Global Catalog        | Búsquedas en todo el forest      |
| 3389      | RDP                   | Escritorio remoto                |
| 5985/5986 | WinRM                 | PowerShell remoto                |

### Archivos críticos

| Archivo              | Ubicación                           | Contenido                                       |
| -------------------- | ----------------------------------- | ----------------------------------------------- |
| **NTDS.dit**         | `C:\Windows\NTDS\` (solo en DCs)    | Base de datos completa de AD (todos los hashes) |
| **SAM**              | `C:\Windows\System32\config\SAM`    | Hashes de cuentas locales                       |
| **SYSTEM**           | `C:\Windows\System32\config\SYSTEM` | Boot key para descifrar SAM                     |
| **LSASS**            | Proceso en memoria                  | Credenciales en caché, tickets Kerberos         |
| **GPP / groups.xml** | SYSVOL                              | Contraseñas en GPOs antiguas (cpassword)        |

### Metodología&#x20;

```
1. RECONOCIMIENTO EXTERNO
   → DNS, subdominios, correos, usuarios expuestos

2. ACCESO INICIAL
   → Password spraying, phishing, exploit de servicio expuesto

3. ENUMERACIÓN INTERNA (con foothold)
   → Usuarios, grupos, equipos, SPNs, ACLs, GPOs, trusts

4. ESCALADA DE PRIVILEGIOS
   → Kerberoasting, AS-REP Roasting, ACL abuse, GPO abuse

5. MOVIMIENTO LATERAL
   → Pass-the-Hash, Pass-the-Ticket, WinRM, RDP, SMB

6. PERSISTENCIA
   → Golden Ticket, Silver Ticket, Skeleton Key, backdoors

7. EXFILTRACIÓN / OBJETIVO FINAL
   → DCSync, volcar NTDS.dit, acceder a recursos críticos
```

> En AD, el mayor valor no es comprometer una máquina individual sino conseguir las credenciales del krbtgt (Golden Ticket) o del Domain Admin: ambos dan control total y persistente del dominio.

> BloodHound es la herramienta más importante para visualizar rutas de ataque en AD. Instalarlo y entender sus queries es el primer paso para cualquier pentest de AD.

> Muchos ataques de AD no requieren explotar vulnerabilidades de software — abusan de configuraciones incorrectas y permisos excesivos que son la norma en entornos reales.
