# FFuF

es un fuzzer web rápido escrito en Go. Se destaca por enumerar rápidamente directorios, archivos y parámetros dentro de aplicaciones web. Su flexibilidad, velocidad y facilidad de uso lo convierten en uno de los favoritos entre los profesionales y entusiastas de la seguridad.

Puedes instalarlo `FFUF` usando el siguiente comando:

```
 go install github.com/ffuf/ffuf/v2@latest
```

#### Casos de uso <a href="#use-cases" id="use-cases"></a>

| `Directory and File Enumeration` | Identifique rápidamente directorios y archivos ocultos en un servidor web.                                       |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `Parameter Discovery`            | Busque y pruebe parámetros dentro de aplicaciones web.                                                           |
| `Brute-Force Attack`             | Realice ataques de fuerza bruta para descubrir credenciales de inicio de sesión u otra información confidencial. |

#### Fuzzing de directorio <a href="#directory-fuzzing" id="directory-fuzzing"></a>

El primer paso es realizar fuzzing de directorios, lo que nos ayuda a descubrir directorios ocultos en el servidor web. Aquí está el comando ffuf que usaremos:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://IP:PORT/FUZZ


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://IP:PORT/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-399
________________________________________________

[...]

w2ksvrus                [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
:: Progress: [220559/220559] :: Job [1/1] :: 100000 req/sec :: Duration: [0:00:03] :: Errors: 0 ::

```

* `-w`: especifica la ruta a la lista de palabras que queremos usar.
* `-u`: especifica la URL base para fuzz.&#x20;
* El `FUZZ` actúa como un marcador de posición donde el fuzzer insertará palabras de la lista de palabras.

#### Fuzzing de archivos <a href="#file-fuzzing" id="file-fuzzing"></a>

El fuzzing de archivos profundiza en el descubrimiento de archivos específicos dentro de esos directorios o incluso en la raíz de la aplicación web.

* `.php`: Archivos que contienen código PHP, un popular lenguaje de programación del lado del servidor.
* `.html`: Archivos que definen la estructura y contenido de las páginas web.
* `.txt`: Archivos de texto sin formato, que a menudo almacenan información o registros simples.
* `.bak`: Los archivos de respaldo se crean para preservar versiones anteriores de los archivos en caso de errores o modificaciones.
* `.js`: Los archivos que contienen código JavaScript agregan interactividad y funcionalidad dinámica a las páginas web.

Utilizar `ffuf` y una lista de palabras de nombres de archivos comunes para buscar archivos ocultos con extensiones específicas:

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://IP:PORT/w2ksvrus/FUZZ -e .php,.html,.txt,.bak,.js -v 


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://IP:PORT/w2ksvrus/FUZZ.html
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Extensions       : .php .html .txt .bak .js 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 111, Words: 2, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/w2ksvrus/dblclk.html
    * FUZZ: dblclk

[Status: 200, Size: 112, Words: 6, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/w2ksvrus/index.html
    * FUZZ: index

:: Progress: [28362/28362] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::

```



<br>
