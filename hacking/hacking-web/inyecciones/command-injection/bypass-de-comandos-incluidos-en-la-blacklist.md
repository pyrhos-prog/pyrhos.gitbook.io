# bypass de comandos incluidos en la blacklist

Hemos discutido varios métodos para eludir los filtros de un solo carácter. Sin embargo, existen diferentes métodos cuando se trata de eludir los comandos incluidos en la lista negra. Una lista negra de comandos generalmente consta de un conjunto de palabras y, si podemos ofuscar nuestros comandos y hacerlos parecer diferentes, es posible que podamos eludir los filtros.

Existen varios métodos de ofuscación de comandos que varían en complejidad, como abordaremos más adelante con las herramientas de ofuscación de comandos. Cubriremos algunas técnicas básicas que pueden permitirnos cambiar la apariencia de nuestro comando para omitir filtros manualmente.

***

### Lista negra de comandos

Hasta ahora hemos omitido con éxito el filtro de caracteres para los caracteres espaciales y puntos y comas en nuestra carga útil. Entonces, volvamos a nuestra primera carga útil y volvamos a agregar el comando `whoami` para ver si se ejecuta:\
![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y una carga útil que contiene un intento de inyección de comando. La sección de respuesta muestra código HTML para un formulario 'Host Checker', lo que permite la entrada de IP y, como resultado, muestra 'Entrada no válida'.](<../../.gitbook/assets/cmdinj_filters_commands_1 (1).jpg>)

Vemos que aunque utilizamos caracteres que no están bloqueados por la aplicación web, la solicitud se bloquea nuevamente una vez que agregamos nuestro comando. Es probable que esto se deba a otro tipo de filtro: un filtro de lista negra de comandos.

Un filtro de lista negra de comandos básico en `PHP` se vería así:

{% code title="example.php" %}
```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```
{% endcode %}

Como podemos ver, verifica cada palabra de la entrada del usuario para ver si coincide con alguna de las palabras incluidas en la lista negra. Sin embargo, este código busca una coincidencia exacta del comando proporcionado, por lo que si enviamos un comando ligeramente diferente, es posible que no se bloquee. Afortunadamente, podemos utilizar varias técnicas de ofuscación que ejecutarán nuestro comando sin utilizar la palabra de comando exacta.

***

### Linux y Windows

Una técnica de ofuscación muy común y sencilla es insertar ciertos caracteres dentro de nuestro comando que generalmente son ignorados por shells de comandos como `bash` o `PowerShell` y ejecutarán el mismo comando como si no estuvieran allí. Algunos de estos caracteres son la comilla simple `'` y la comilla doble `"`, además de algunos otros.

Las más fáciles de usar son las comillas y funcionan tanto en servidores Linux como Windows. Por ejemplo, si queremos ofuscar el comando `whoami`, podemos insertar comillas simples entre sus caracteres, de la siguiente manera:

{% code title="shell-session" %}
```shell-session
21y4d@htb[/htb]$ w'h'o'am'i

21y4d
```
{% endcode %}

Lo mismo funciona también con comillas dobles:

{% code title="shell-session" %}
```shell-session
21y4d@htb[/htb]$ w"h"o"am"i

21y4d
```
{% endcode %}

{% hint style="info" %}
Importante: no podemos mezclar tipos de comillas y el número de comillas debe ser par (even). Si se mezclan o quedan sin cerrar, la expresión será distinta y puede fallar.
{% endhint %}

Podemos probar uno de los anteriores en nuestra carga útil (`127.0.0.1%0aw'h'o'am'i`) y ver si funciona:

**Solicitud POST de ejemplo**

![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y un intento de inyección de comando. La sección de respuesta muestra HTML para un formulario 'Host Checker', lo que permite la entrada de IP y muestra resultados de ping para 127.0.0.1.](<../../.gitbook/assets/cmdinj_filters_commands_2 (1).jpg>)

Como podemos ver, este método realmente funciona.

***

### Solo Linux

Podemos insertar algunos otros caracteres exclusivos de Linux en medio de los comandos y `bash` shell los ignoraría y ejecutaría el comando. Estos caracteres incluyen la barra invertida `\` y el carácter del parámetro posicional `$@`. Esto funciona exactamente como lo hacía con las comillas, pero en este caso, el número de caracteres no tiene que ser par, y podemos insertar solo uno de ellos si queremos:

{% code title="bash" %}
```bash
who$@ami
w\ho\am\i
```
{% endcode %}

Ejercicio: Pruebe los dos ejemplos anteriores en su carga útil y vea si funcionan para omitir el filtro de comando. Si no lo hacen, esto puede indicar que es posible que haya utilizado un carácter filtrado. ¿Podrías evitar eso también, utilizando las técnicas que aprendimos en la sección anterior?

<details>

<summary>Ejercicio — pistas y recordatorios</summary>

* Revise si el filtro bloquea caracteres como `'`, `"`, `\` o `$`.
* Si alguno de estos caracteres está filtrado, aplique técnicas de evasión de filtros de caracteres (por ejemplo, URL-encoding, usar otras formas de escape o concatenación dependiente del shell).
* Asegúrese de que las comillas estén balanceadas y no mezcle tipos de comillas cuando lo haga.

</details>

***

### Solo Windows

También hay algunos caracteres exclusivos de Windows que podemos insertar en medio de comandos que no afectan el resultado, como el carácter de escape/caret `^`, como podemos ver en el siguiente ejemplo:

{% code title="cmd-session" %}
```cmd-session
C:\htb> who^ami

21y4d
```
{% endcode %}

En la siguiente sección, analizaremos algunas técnicas más avanzadas para la ofuscación de comandos y la omisión de filtros.
