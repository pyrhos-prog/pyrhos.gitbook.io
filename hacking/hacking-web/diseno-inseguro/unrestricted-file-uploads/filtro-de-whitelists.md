# Filtro de whitelists

Como se analizó en la sección anterior, el otro tipo de validación de extensión de archivo es mediante el uso de una whitelist of allowed file extensions. Una lista blanca es generalmente más segura que una lista negra. El servidor web sólo permitiría las extensiones especificadas y no sería necesario que la lista fuera exhaustiva para cubrir extensiones poco comunes.

Aún así, existen diferentes casos de uso para una lista negra y para una lista blanca. Una lista negra puede ser útil en los casos en que la funcionalidad de carga debe permitir una amplia variedad de tipos de archivos (por ejemplo, Administrador de archivos), mientras que una lista blanca generalmente solo se usa con funcionalidades de carga donde solo se permiten unos pocos tipos de archivos. Ambos también pueden utilizarse en conjunto.

***

### Extensiones de lista blanca

Comencemos el ejercicio al final de esta sección e intentemos cargar una extensión PHP poco común, como `.phtml`, y ver si todavía podemos subirlo como lo hicimos en la sección anterior:

http://SERVER\_IP:PORT/

![Aviso de actualización de imagen de perfil con botón de carga, solo se permiten imágenes.](/broken/files/5fc1b106ee8a68a94c1fb41d521ca806da0f0130)

Vemos que recibimos un mensaje que dice `Only images are allowed`, lo que puede ser más común en aplicaciones web que ver un tipo de extensión bloqueado. Sin embargo, los mensajes de error no siempre reflejan qué forma de validación se está utilizando, así que intentemos buscar extensiones permitidas como lo hicimos en la sección anterior, usando la misma lista de palabras que usamos anteriormente:

http://SERVER\_IP:PORT/

![Solicitudes HTTP con varias cargas útiles PHP, todas devuelven el estado 200, solo se permiten imágenes.](../../.gitbook/assets/file_uploads_whitelist_fuzz.jpg)

Podemos ver que todas las variaciones de las extensiones PHP están bloqueadas (por ejemplo `php5`, `php7`, `phtml`). Sin embargo, la lista de palabras que utilizamos también contenía otras extensiones "maliciosas" que no fueron bloqueadas y se cargaron correctamente. Entonces, intentemos entender cómo pudimos cargar estas extensiones y en qué casos podremos utilizarlas para ejecutar código PHP en el servidor back-end.

El siguiente es un ejemplo de una prueba de lista blanca de extensiones de archivo:

{% code title="Validación con regex" %}
```php
$fileName = basename($_FILES["uploadFile"]["name"]);
if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
    echo "Only images are allowed";
    die();
}
```
{% endcode %}

Vemos que el script utiliza una expresión regular (`regex`) para probar si el nombre del archivo contiene alguna extensión de imagen incluida en la lista blanca. La cuestión aquí radica en el `regex`, ya que solo verifica si el nombre del archivo `contains` la extensión y no si realmente `ends` con él. Muchos desarrolladores cometen estos errores debido a una comprensión débil de los patrones de expresiones regulares.

Entonces, veamos cómo podemos omitir estas pruebas para cargar scripts PHP.

***

### Extensiones dobles

El código solo prueba si el nombre del archivo contiene una extensión de imagen; un método sencillo para pasar la prueba de expresiones regulares es a través de Double Extensions. Por ejemplo, si el `.jpg` se permitió la extensión, podemos agregarla en nuestro nombre de archivo cargado y aún así finalizar nuestro nombre de archivo con `.php` (p. ej. `shell.jpg.php`), en cuyo caso deberíamos poder pasar la prueba de lista blanca, mientras seguimos cargando un script PHP que pueda ejecutar código PHP.

Ejercicio: Intente difuminar el formulario de carga con Esta lista de palabras para encontrar qué extensiones están incluidas en la lista blanca del formulario de carga.

Interceptemos una solicitud de carga normal y modifiquemos el nombre del archivo a `shell.jpg.php`, y modificar su contenido al de un shell web:

http://SERVER\_IP:PORT/

![Solicitud POST a /upload.php con archivo PHP disfrazado de imagen, nombre de archivo 'shell.jpg.php'.](../../.gitbook/assets/file_uploads_double_ext_request.jpg)

Ahora, si visitamos el archivo cargado e intentamos enviar un comando, podemos ver que efectivamente ejecuta con éxito los comandos del sistema, lo que significa que el archivo que cargamos es un script PHP completamente funcional:

http://SERVER\_IP:PORT/profile\_images/shell.jpg.php?cmd=id

![UID, GID y grupos configurados en 33 (datos www).](../../.gitbook/assets/file_uploads_php_manual_shell.jpg)

Sin embargo, es posible que esto no siempre funcione, ya que algunas aplicaciones web pueden utilizar un método estricto `regex` patrón, como se mencionó anteriormente, como el siguiente:

{% code title="Validación estricta con regex" %}
```php
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) {
    ...SNIP...
}
```
{% endcode %}

Este patrón solo debe considerar la extensión de archivo final, tal como la utiliza (`^.*\.`) para hacer coincidir todo hasta el final (`.`), y luego usa (`$`) al final para que solo coincidan las extensiones que terminan el nombre del archivo. Entonces, el above attack would not work. Sin embargo, algunas técnicas de explotación pueden permitirnos eludir este patrón, pero la mayoría se basan en configuraciones erróneas o sistemas obsoletos.

***

### Extensión doble inversa

En algunos casos, la funcionalidad de carga de archivos en sí puede no ser vulnerable, pero la configuración del servidor web puede generar una vulnerabilidad. Por ejemplo, una organización puede utilizar una aplicación web de código abierto, que tiene una funcionalidad de carga de archivos. Incluso si la funcionalidad de carga de archivos utiliza un patrón de expresión regular estricto que solo coincide con la extensión final en el nombre del archivo, la organización puede utilizar configuraciones inseguras para el servidor web.

Por ejemplo, el `/etc/apache2/mods-enabled/php7.4.conf` para el `Apache2` servidor web puede incluir la siguiente configuración:

{% code title="FilesMatch en Apache" %}
```xml
<FilesMatch ".+\.ph(ar|p|tml)">
    SetHandler application/x-httpd-php
</FilesMatch>
```
{% endcode %}

La configuración anterior es cómo el servidor web determina qué archivos permiten la ejecución de código PHP. Especifica una lista blanca con un patrón de expresión regular que coincide `.phar`, `.php`, y `.phtml`. Sin embargo, este patrón de expresión regular puede tener el mismo error que vimos antes si olvidamos terminarlo con (`$`). En tales casos, a cualquier archivo que contenga las extensiones anteriores se le permitirá la ejecución de código PHP, incluso si no termina con la extensión PHP. Por ejemplo, el nombre del archivo `shell.php.jpg` debe pasar la prueba de lista blanca anterior ya que termina con `.jpg`, y podría ejecutar código PHP debido a la configuración incorrecta anterior, ya que contiene `.php` en su nombre.

Ejercicio: La aplicación web aún puede utilizar una lista negra para rechazar solicitudes que contengan `PHP` extensiones. Intente fusionar el formulario de carga con el Lista de palabras PHP para encontrar qué extensiones están en la lista negra del formulario de carga.

Intentemos interceptar una solicitud de carga de imagen normal y usemos el nombre de archivo anterior para pasar la estricta prueba de lista blanca:

http://SERVER\_IP:PORT/

![Solicitud POST a /upload.php con el archivo 'shell.php.jpg' disfrazado de imagen.](../../.gitbook/assets/file_uploads_reverse_double_ext_request.jpg)

Ahora podemos visitar el archivo cargado e intentar ejecutar un comando:

http://SERVER\_IP:PORT/profile\_images/shell.php.jpg?cmd=id

![UID, GID y grupos configurados en 33 (datos www).](../../.gitbook/assets/file_uploads_php_manual_shell.jpg)

Como podemos ver, evitamos con éxito la estricta prueba de lista blanca y explotamos la mala configuración del servidor web para ejecutar código PHP y obtener control sobre el servidor.

***

### Inyección de caracteres

Por último, analicemos otro método para evitar una prueba de validación de lista blanca a través de Character Injection. Podemos inyectar varios caracteres antes o después de la extensión final para provocar que la aplicación web malinterprete el nombre del archivo y ejecute el archivo cargado como un script PHP.

Los siguientes son algunos de los caracteres que podemos intentar inyectar:

* `%20`
* `%0a`
* `%00`
* `%0d0a`
* `/`
* `.\`
* `.`
* `…`
* `:`

Cada carácter tiene un caso de uso específico que puede engañar a la aplicación web para que malinterprete la extensión del archivo. Por ejemplo, `shell.php%00.jpg` funciona con servidores PHP con versión `5.X` o antes, ya que hace que el servidor web PHP finalice el nombre del archivo después de (`%00`), y guárdelo como `shell.php`, sin dejar de pasar la lista blanca. Lo mismo se puede utilizar con aplicaciones web alojadas en un servidor Windows inyectando dos puntos (`:`) antes de la extensión de archivo permitida (p. ej. `shell.aspx:.jpg`), que también debería escribir el archivo como `shell.aspx`. De manera similar, cada uno de los otros caracteres tiene un caso de uso que puede permitirnos cargar un script PHP sin pasar por la prueba de validación de tipo.

Podemos escribir un pequeño script bash que genere todas las permutaciones del nombre del archivo, donde los caracteres anteriores se inyectarían antes y después de ambos `PHP` y `JPG` extensiones, como sigue:

{% code title="Generador de permutaciones (bash)" %}
```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
       echo "shell$char$ext.jpg" >> wordlist.txt
       echo "shell$ext$char.jpg" >> wordlist.txt
       echo "shell.jpg$char$ext" >> wordlist.txt
       echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```
{% endcode %}

Con esta lista de palabras personalizada, podemos ejecutar un escaneo fuzzing con `Burp Intruder`, similar a los que hicimos antes. Si el back-end o el servidor web están desactualizados o tienen ciertas configuraciones incorrectas, algunos de los nombres de archivos generados pueden eludir la prueba de lista blanca y ejecutar código PHP.

***

{% stepper %}
{% step %}
### Ejercicio 1

Intente difuminar el formulario de carga con esta lista de palabras: https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt

Objetivo: identificar qué extensiones están incluidas en la lista blanca del formulario de carga.
{% endstep %}

{% step %}
### Ejercicio 2

La aplicación web aún puede utilizar una lista negra para rechazar extensiones PHP. Use esta lista: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst

Objetivo: encontrar qué extensiones están en la lista negra del formulario de carga y probar ataques de doble extensión (`shell.php.jpg`, `shell.jpg.php`).
{% endstep %}

{% step %}
### Ejercicio 3

Amplíe el script de generación de permutaciones para añadir más extensiones PHP y caracteres, genere una wordlist y fuzzee el formulario de carga (por ejemplo, con Burp Intruder).

Objetivo: encontrar nombres de archivo que pasen la validación y, tras la carga, permitan ejecutar código PHP debido a errores de validación o mala configuración del servidor.
{% endstep %}
{% endstepper %}

***

{% hint style="info" %}
Notas importantes:

* La efectividad de cada técnica depende de la validación a nivel de aplicación y de la configuración del servidor web.
* Algunas técnicas solo funcionan contra versiones antiguas de PHP o servidores mal configurados.
* No agregue información ni datos que no estén presentes en el contenido original cuando realice pruebas.
{% endhint %}
