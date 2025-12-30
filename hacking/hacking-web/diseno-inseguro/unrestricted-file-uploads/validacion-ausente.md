# Validación ausente

El tipo más básico de vulnerabilidad de carga de archivos ocurre cuando la aplicación web `does not have any form of validation filters` en los archivos cargados, permitiendo la carga de cualquier tipo de archivo de forma predeterminada.

Con este tipo de aplicaciones web vulnerables, podemos cargar directamente nuestro shell web o script de shell inverso a la aplicación web y luego, con solo visitar el script cargado, podemos interactuar con nuestro shell web o enviar el shell inverso.

***

### Carga arbitraria de archivos

Comencemos el ejercicio al final de esta sección y veremos un `Employee File Manager` aplicación web, que nos permite cargar archivos personales a la aplicación web:

![Área de carga de archivos para el archivo de empleados con opción de arrastrar y soltar o hacer clic y un botón 'Cargar'.](../../.gitbook/assets/file_uploads_file_manager.jpg)

La aplicación web no menciona nada sobre qué tipos de archivos están permitidos, y podemos arrastrar y soltar cualquier archivo que queramos, y su nombre aparecerá en el formulario de carga, incluido `.php` fișieri:

![Área de carga de archivos para el archivo de empleados con 'shell.php' seleccionado, tamaño 1 kb y un botón 'Cargar'.](../../.gitbook/assets/file_uploads_file_selected_php_file.jpg)

Además, si hacemos clic en el formulario para seleccionar un archivo, el cuadro de diálogo del selector de archivos no especifica ningún tipo de archivo, como dice `All Files` para el tipo de archivo, lo que también puede sugerir que no se especifica ningún tipo de restricciones o limitaciones para la aplicación web:

![Cuadro de diálogo de carga de archivos que muestra 'shell.php' seleccionado, tamaño 1,3 kB, tipo Programa, con opciones 'Abrir' y 'Cargar'.](../../.gitbook/assets/file_uploads_file_selection_dialog.jpg)

Todo esto nos dice que el programa parece no tener restricciones de tipo de archivo en el front-end, y si no se especificaran restricciones en el back-end, podríamos cargar tipos de archivos arbitrarios al servidor back-end para obtener control total sobre él.

***

### Identificación de un marco web

Necesitamos cargar un script malicioso para probar si podemos cargar cualquier tipo de archivo al servidor back-end y probar si podemos usarlo para explotar el servidor back-end. Muchos tipos de scripts pueden ayudarnos a explotar aplicaciones web mediante la carga arbitraria de archivos, más comúnmente un `Web Shell` guión y un `Reverse Shell` guion.

Un Web Shell nos proporciona un método sencillo para interactuar con el servidor back-end aceptando comandos de shell e imprimiendo su salida dentro del navegador web. Un shell web debe escribirse en el mismo lenguaje de programación que ejecuta el servidor web, ya que ejecuta funciones y comandos específicos de la plataforma para ejecutar comandos del sistema en el servidor back-end, lo que hace que los shells web no sean scripts multiplataforma. Entonces, el primer paso sería identificar qué idioma ejecuta la aplicación web.

Esto suele ser relativamente simple, ya que a menudo podemos ver la extensión de la página web en las URL, lo que puede revelar el lenguaje de programación que ejecuta la aplicación web. Sin embargo, en ciertos marcos web y lenguajes web, `Web Routes` se utilizan para asignar URL a páginas web, en cuyo caso es posible que no se muestre la extensión de la página web. Además, la explotación de la carga de archivos también sería diferente, ya que es posible que nuestros archivos cargados no sean enrutables o accesibles directamente.

Un método sencillo para determinar qué idioma ejecuta la aplicación web es visitar el `/index.ext` página, donde cambiaríamos `ext` con varias extensiones web comunes, como `php`, `asp`, `aspx`, entre otros, para ver si alguno de ellos existe.

Por ejemplo, cuando visitamos nuestro ejercicio a continuación, vemos su URL como `http://SERVER_IP:PORT/`, como el `index` La página generalmente está oculta de forma predeterminada. Pero, si intentamos visitar `http://SERVER_IP:PORT/index.php`, obtendríamos la misma página, lo que significa que esto es realmente un `PHP` aplicación web. Por supuesto, no necesitamos hacer esto manualmente, ya que podemos usar una herramienta como Burp Intruder para difuminar la extensión del archivo usando un [Extensiones web](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) lista de palabras, como veremos en las próximas secciones. Sin embargo, es posible que este método no siempre sea preciso, ya que es posible que la aplicación web no utilice páginas de índice o que utilice más de una extensión web.

Varias otras técnicas pueden ayudar a identificar las tecnologías que ejecutan la aplicación web, como el uso de [Wappalyzer](https://www.wappalyzer.com/) extensión, que está disponible para todos los navegadores principales. Una vez agregado a nuestro navegador, podemos hacer clic en su ícono para ver todas las tecnologías que ejecutan la aplicación web:

![Área de carga de archivos con superposición de Wappalyzer que muestra tecnologías: Apache, PHP, Ubuntu, cdnjs, jQuery.](../../.gitbook/assets/file_uploads_wappalyzer.jpg)

Como podemos ver, la extensión no solo nos dijo que la aplicación web se ejecuta en `PHP`, pero también identificó el tipo y la versión del servidor web, el sistema operativo back-end y otras tecnologías en uso. Estas extensiones son esenciales en el arsenal de un probador de penetración web, aunque siempre es mejor conocer métodos manuales alternativos para identificar el marco web, como el método anterior que analizamos.

También podemos ejecutar escáneres web para identificar el marco web, como escáneres Burp/ZAP u otras herramientas de evaluación de vulnerabilidades web. Al final, una vez que identificamos el idioma que ejecuta la aplicación web, podemos cargar un script malicioso escrito en el mismo idioma para explotar la aplicación web y obtener control remoto sobre el servidor back-end.

{% stepper %}
{% step %}
### Método 1 — Probar extensiones comunes

Visitar manualmente o automatizar peticiones a `/index.ext` con extensiones comunes (por ejemplo: `php`, `asp`, `aspx`) para ver si alguna devuelve la misma página y así identificar el lenguaje/tecnología.
{% endstep %}

{% step %}
### Método 2 — Usar extensiones o herramientas

Usar herramientas como la extensión Wappalyzer para identificar tecnologías en uso (servidor web, lenguaje, librerías). También se pueden usar escáneres web (Burp/ZAP) para obtener más información automatizada.
{% endstep %}
{% endstepper %}

***

### Identificación de vulnerabilidades

Ahora que hemos identificado el marco web que ejecuta la aplicación web y su lenguaje de programación, podemos probar si podemos cargar un archivo con la misma extensión. Como prueba inicial para identificar si podemos cargar archivos arbitrarios `PHP` archivos, vamos a crear uno básico `Hello World` script para probar si podemos ejecutar `PHP` código con nuestro archivo cargado.

Para ello escribiremos:

```php
<?php echo "Hello HTB";?>
```

a `test.php`, e intenta subirlo a la aplicación web:

![Archivo cargado exitosamente con el botón 'Descargar archivo'.](../../.gitbook/assets/file_uploads_upload_php.jpg)

El archivo parece haberse cargado correctamente, ya que recibimos un mensaje que dice `File successfully uploaded`, lo que significa que `the web application has no file validation whatsoever on the back-end`. Ahora podemos hacer clic en `Download` botón, y la aplicación web nos llevará a nuestro archivo cargado:

![Texto: Hola HTB](../../.gitbook/assets/file_uploads_hello_htb.jpg)

Como podemos ver, la página imprime nuestro `Hello HTB` mensaje, lo que significa que el `echo` Se ejecutó la función para imprimir nuestra cadena y la ejecutamos correctamente `PHP` código en el servidor back-end. Si la página no pudiera ejecutar código PHP, veríamos nuestro código fuente impreso en la página.

En la siguiente sección, veremos cómo explotar esta vulnerabilidad para ejecutar código en el servidor back-end y tomar el control sobre él.
