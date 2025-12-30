# Filtros de blacklists

### Extensiones de lista negra

Comencemos probando una de las omisiones del lado del cliente que aprendimos en la sección anterior para cargar un script PHP al servidor back-end. Interceptaremos una solicitud de carga de imágenes con Burp, reemplazaremos el contenido y el nombre del archivo con nuestros scripts PHP y reenviamos la solicitud:

![Actualizar imagen de perfil con botón de carga; extensión no permitida.](../../.gitbook/assets/file_uploads_disallowed_type.jpg)

Como podemos ver, nuestro ataque no tuvo éxito esta vez, ya que lo logramos `Extension not allowed`. Esto indica que la aplicación web puede tener algún tipo de validación de tipo de archivo en el back-end, además de las validaciones del front-end.

Generalmente existen dos formas comunes de validar una extensión de archivo en el back-end:

* Prueba contra a `blacklist` de tipos
* Prueba contra a `whitelist` de tipos

Además, la validación también puede comprobar el `file type` o el `file content` para coincidencia de tipos. La forma más débil de validación entre ellas es `testing the file extension against a blacklist of extension` para determinar si la solicitud de carga debe bloquearse. Por ejemplo, el siguiente fragmento de código verifica si la extensión del archivo cargado es `PHP` y elimina la solicitud si es:

{% code title="Comprobación de lista negra (ejemplo)" %}
```php
$fileName = basename($_FILES["uploadFile"]["name"]);
$extension = pathinfo($fileName, PATHINFO_EXTENSION);
$blacklist = array('php', 'php7', 'phps');

if (in_array($extension, $blacklist)) {
    echo "File type not allowed";
    die();
}
```
{% endcode %}

El código toma la extensión del archivo (`$extension`) del nombre del archivo cargado (`$fileName`) y luego compararlo con una lista de extensiones incluidas en la lista negra (`$blacklist`). Sin embargo, este método de validación tiene un defecto importante. `It is not comprehensive`, ya que muchas otras extensiones no están incluidas en esta lista, que aún pueden usarse para ejecutar código PHP en el servidor back-end si se cargan.

{% hint style="info" %}
La comparación anterior también distingue entre mayúsculas y minúsculas y solo considera extensiones en minúsculas. En los servidores Windows, los nombres de los archivos no distinguen entre mayúsculas y minúsculas, por lo que podemos intentar cargar un `php` con un caso mixto (por ejemplo `pHp`), que también puede eludir la lista negra y aún así debería ejecutarse como un script PHP.
{% endhint %}

Entonces, intentemos explotar esta debilidad para evitar la lista negra y cargar un archivo PHP.

***

### Extensiones de fuzzing

Como la aplicación web parece estar probando la extensión del archivo, nuestro primer paso es combinar la funcionalidad de carga con una lista de extensiones potenciales y ver cuáles de ellas devuelven el mensaje de error anterior. Cualquier solicitud de carga que no devuelva un mensaje de error, devuelva un mensaje diferente o logre cargar el archivo puede indicar una extensión de archivo permitida.

Hay muchas listas de extensiones que podemos utilizar en nuestro escaneo fuzzing. `PayloadsAllTheThings` proporciona listas de extensiones para [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) y [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) aplicaciones web. También podemos utilizar `SecLists` lista de comunes [Extensiones web](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt).

Podemos utilizar cualquiera de las listas anteriores para nuestro escaneo de fuzzing. Mientras probamos una aplicación PHP, descargaremos y usaremos la anterior lista [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst). A continuación se muestra un flujo de trabajo típico con Burp Intruder para buscar extensiones permitidas:

{% stepper %}
{% step %}
### Preparar la petición en Intruder

Desde `Burp History`, localiza la última solicitud para `/upload.php`, haz clic derecho sobre ella y selecciona `Send to Intruder`.\
En la pestaña `Positions`, `Clear` cualquier posición establecida automáticamente.
{% endstep %}

{% step %}
### Seleccionar la posición a fuzzear

Selecciona la extensión en `filename="HTB.php"` (por ejemplo, el `.php`) y haz clic en `Add` para agregarla como posición de fuzzing. Conserva el contenido del archivo para este ataque, ya que solo interesa difuminar las extensiones.
{% endstep %}

{% step %}
### Cargar la lista de extensiones

En la pestaña `Payloads`, carga la lista de extensiones PHP descargada anteriormente en `Payload Options`.\
Desmarca `URL Encoding` para evitar codificar el (`.`) antes de la extensión del archivo.
{% endstep %}

{% step %}
### Iniciar el ataque y analizar resultados

Haz clic en `Start Attack` para comenzar. Ordena los resultados por `Length` y busca respuestas que difieran (por ejemplo, una respuesta con `File successfully uploaded` o un tamaño distinto). Esas respuestas indican extensiones que no están en la lista negra.
{% endstep %}
{% endstepper %}

Resultado ilustrado:

![Solicitud HTTP POST a /upload.php con archivo HTB$.php$, tipo de ataque Sniper.](../../.gitbook/assets/file_uploads_burp_fuzz_extension.jpg)

![Tabla de cargas útiles con estado HTTP 200, archivo cargado exitosamente.](../../.gitbook/assets/file_uploads_burp_intruder_result.jpg)

Podemos ordenar los resultados por `Length`, y veremos que todas las solicitudes con la longitud del contenido (`193`) pasaron la validación de la extensión, ya que todos respondieron con `File successfully uploaded`. Por el contrario, el resto respondió con un mensaje de error que decía `Extension not allowed`.

***

### Extensiones no incluidas en la lista negra

Ahora, podemos intentar cargar un archivo usando cualquiera de las `allowed extensions` desde arriba, y algunos de ellos pueden permitirnos ejecutar código PHP. `Not all extensions will work with all web server configurations`, por lo que es posible que necesitemos probar varias extensiones para obtener una que ejecute código PHP con éxito.

Usemos la `.phtml` extensión, que los servidores web PHP a menudo permiten para ejecución de código. Podemos hacer clic derecho en su solicitud en los resultados de Intruder y seleccionar `Send to Repeater`. Ahora, todo lo que tenemos que hacer es repetir lo que hemos hecho en las dos secciones anteriores cambiando el nombre del archivo para usar la `.phtml` extensión y cambiar el contenido al de un shell web PHP:

![Solicitud HTTP POST a /upload.php con archivo shell.phtml, respuesta 200 OK, archivo cargado exitosamente.](../../.gitbook/assets/file_uploads_php5_web_shell.jpg)

Como podemos ver, nuestro archivo parece haber sido efectivamente cargado. El paso final es visitar nuestro archivo de carga, que debe estar en el directorio de carga de imágenes (`profile_images`), como vimos en el apartado anterior. Luego, podemos probar la ejecución de un comando, que debería confirmar que evitamos con éxito la lista negra y cargamos nuestro shell web:

![UID, GID y grupos configurados en 33 (www-data).](../../.gitbook/assets/file_uploads_php_manual_shell.jpg)
