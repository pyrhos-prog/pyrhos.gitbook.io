---
icon: building-magnifying-glass
---

# SMB

es un protocolo cliente-servidor utilizado para compartir archivos, directorios, impresoras y otros recursos en red. Opera principalmente sobre **TCP 445** (SMB directo) y, en entornos con NetBIOS, sobre **137–139**.

En sistemas Unix/Linux se implementa mediante **Samba**, que puede funcionar como servidor standalone o miembro de dominio e incluso como controlador de dominio en versiones modernas.

### Que buscamos en cada caso

SMB standalone (Linux Samba / Windows sin dominio):

* Enumeras usuarios locales
* Shares
* Permisos

SMB en Active Directory:

* Enumeras usuarios dominio
* Grupos privilegiados
* Controladores de dominio

### Indicadores de mala administración

* SMB signing disabled
* SMBv1 enabled
* Shares con WRITE
* Null session habilitada
* Password policy débil
* Usuarios sin expiración
* Scripts de login accesibles

## Nmap

```bash
nmap -p139,445 -sS -sV -Pn <IP>
```

* `-sS` → SYN scan
* `-sV` → version detection
* `-Pn` → omitir ICMP

### Scripts NSE específicos SMB

```bash
nmap --script smb-protocols -p445 <IP>
```

Devuelve:

* SMBv1 habilitado
* SMB2, SMB3
* Dialectos soportados

Relevancia:

* SMBv1 → posible explotación tipo EternalBlue
* SMB signing requerido o no

```bash
nmap --script smb-security-mode -p445 <IP>
```

Extrae:

* Authentication level
* SMB signing required/optional/disabled

Interpretación:

* Signing disabled → posible relay attack

```bash
nmap --script smb-enum-shares -p445 <IP>
```

Lista:

* Shares accesibles
* Permisos detectados

```bash
nmap --script smb-enum-users -p445 <IP>
```

Requiere null session o credenciales válidas.

```bash
nmap --script smb-enum-domains -p445 <IP>
```

Detecta:

* Dominio
* SID base
* Controlador de dominio

#### smb-os-discovery

```bash
nmap --script smb-os-discovery -p445 <IP>
```

Obtiene:

* OS exacto
* Nombre NetBIOS
* Nombre dominio
* FQDN
* Arquitectura

## smbclient – Interacción directa

Es cliente SMB tipo FTP.

### Listar shares

```bash
smbclient -L //<IP> -N
```

Parámetros:

* `-L` → listar shares
* `-N` → sin contraseña
* `-U user` → usuario
* `-W domain` → dominio
* `-p 445` → puerto

Con credenciales:

```bash
smbclient -L //<IP> -U user%password
```

### Conexión a share

```bash
smbclient //<IP>/share -U user
```

**Consejos de enumeración:**

* Buscar backups
* Buscar scripts .ps1, .bat
* Buscar archivos .kdbx
* Buscar credenciales en texto plano

### Forzar SMB1

```bash
smbclient //<IP>/share -m SMB1
```

Útil si el servidor rechaza dialectos modernos.

***

## rpcclient – Enumeración RPC avanzada

Permite interacción con SAMR y LSA.

### Conexión anónima

```bash
rpcclient -U "" -N <IP>
```

### Enumeración básica

Dentro de rpcclient:

```
srvinfo
```

Información:

* OS
* Build
* Roles

```
enumdomains
```

Lista dominios/workgroups.

```
querydominfo
```

Devuelve:

* SID
* Número usuarios
* Política básica

### Enumeración de usuarios

```
enumdomusers
```

Devuelve:

* RID
* Username

```
queryuser <RID>
```

Devuelve:

* Last logon
* Password last set
* Account flags

### Enumeración de grupos

```
enumdomgroups
querygroup <RID>
```

Permite identificar:

* Domain Admins
* Enterprise Admins
* Backup Operators

## &#x20;RID Cycling

Técnica para descubrir usuarios cuando enumdomusers está bloqueado.

Base:\
SID + RID incremental.

Ejemplo automatizado:

```bash
lookupsid.py anonymous@<IP>
```

O manual:

```bash
rpcclient -N -U "" <IP> -c "lsaquery"
```

Obtener SID base → iterar RIDs.

RID comunes:

* 500 → Administrator
* 501 → Guest
* 512 → Domain Admins
* 513 → Domain Users

## &#x20;smbmap – Permisos detallados

```bash
smbmap -H <IP>
```

Parámetros:

* `-u user`
* `-p password`
* `-d domain`
* `-r share` → listar recursivamente
* `-x command` → ejecutar comando remoto (si posible)

Ejemplo:

```bash
smbmap -H <IP> -u user -p pass -r shared -A password
```

* `-A` → buscar string específica

Devuelve:

* READ
* WRITE
* NO ACCESS

Interpretación:\
WRITE → posible subida de webshell o backdoor.

## CrackMapExec (CME)

Framework post-explotación y enumeración masiva.

### Escaneo básico

```bash
crackmapexec smb <IP>
```

Devuelve:

* OS
* Signing
* SMBv1
* Dominio

### Enumerar shares

```bash
crackmapexec smb <IP> --shares
```

### Validar credenciales

```bash
crackmapexec smb <IP> -u user -p password
```

### Password spraying

```bash
crackmapexec smb <IP_range> -u users.txt -p 'Password123'
```

### Dump de usuarios (con credenciales)

```bash
crackmapexec smb <IP> --users
```

### Dump SAM (requiere privilegios)

```bash
crackmapexec smb <IP> --sam
```

## enum4linux-ng

Automatización completa.

```bash
enum4linux-ng.py <IP> -A
```

Flags importantes:

* `-A` → all
* `-U` → usuarios
* `-G` → grupos
* `-S` → shares
* `-P` → política contraseña

Ventaja:\
Combina:

* rpcclient
* smbclient
* nmblookup
* LDAP básico

## Enumeración con credenciales válidas

Si tienes credenciales de dominio:

```bash
crackmapexec smb <IP> -u user -p pass -d domain
```

Permite:

* Acceso a shares ocultos
* Enumeración de GPO
* Dump de hashes
* Movimiento lateral

