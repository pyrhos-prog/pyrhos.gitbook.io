# bypass de filtros de espaciados

Existen numerosas formas de detectar intentos de inyección y múltiples métodos para evitar estas detecciones. Demostraremos el concepto de detección y cómo funciona la omisión usando Linux como ejemplo. Aprenderemos a utilizar estos bypasses y eventualmente podremos prevenirlos. Una vez que comprendamos bien cómo funcionan, podremos consultar varias fuentes en Internet para descubrir otros tipos de bypasses y aprender cómo mitigarlos.

***

## Evite a los operadores incluidos en la lista negra

Veremos que la mayoría de los operadores de inyección están efectivamente en la lista negra. Sin embargo, el carácter de nueva línea generalmente no está en la lista negra, ya que puede ser necesario en la carga útil en sí. Sabemos que el carácter de nueva línea funciona al agregar nuestros comandos tanto en Linux como en Windows, así que intentemos usarlo como nuestro operador de inyección:

![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP configurada en '127.0.0.1%0a'. La respuesta muestra HTML para un formulario de Host Checker y resultados de ping para 127.0.0.1.](<../../.gitbook/assets/cmdinj_filters_operator (1).jpg>)

Como podemos ver, aunque nuestra carga útil incluía un carácter de nueva línea, nuestra solicitud no fue denegada y obtuvimos el resultado del comando ping — lo que significa que este carácter no está en la lista negra, y podemos usarlo como nuestro operador de inyección. Comencemos discutiendo cómo evitar un carácter comúnmente incluido en la lista negra: un carácter espacial.

***

## Evite los espacios incluidos en la lista negra

Ahora que tenemos un operador de inyección que funciona, modifiquemos nuestra carga útil original y enviémosla nuevamente como (`127.0.0.1%0a whoami`):

![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP configurada en '127.0.0.1%0a+whoami'. La respuesta muestra HTML para un formulario de Host Checker y un mensaje de 'Entrada no válida'.](<../../.gitbook/assets/cmdinj_filters_spaces_1 (1).jpg>)

Como podemos ver, todavía obtenemos un mensaje de `invalid input`, lo que significa que todavía tenemos otros filtros que omitir. Entonces, como hicimos antes, agreguemos solo el siguiente carácter (que es un espacio) y veamos si causó la solicitud denegada:

![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP configurada en '127.0.0.1%0a+whoami'. La respuesta muestra HTML para un formulario de Host Checker y un mensaje de 'Entrada no válida'.](<../../.gitbook/assets/cmdinj_filters_spaces_2 (1).jpg>)

Como podemos ver, el carácter espacial también está en la lista negra. Un espacio es un carácter comúnmente incluido en la lista negra, especialmente si la entrada no debe contener ningún espacio (por ejemplo, una IP). Aún así, ¡hay muchas formas de agregar un carácter espacial sin usar realmente el carácter espacial!

{% stepper %}
{% step %}
### Uso de pestañas

Usar pestañas (%09) en lugar de espacios es una técnica que puede funcionar, ya que tanto Linux como Windows aceptan comandos con pestañas entre argumentos y se ejecutan de la misma manera. Entonces, intentemos usar una pestaña en lugar del carácter de espacio (`127.0.0.1%0a%09`) y ver si nuestra solicitud es aceptada:

![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP configurada en '127.0.0.1%0a%09'. La respuesta muestra HTML para un formulario de Host Checker y resultados de ping para 127.0.0.1.](<../../.gitbook/assets/cmdinj_filters_spaces_3 (1).jpg>)

Como podemos ver, evitamos con éxito el filtro de caracteres espaciales utilizando una pestaña en su lugar.
{% endstep %}

{% step %}
### Usando $IFS

El uso de la variable de entorno de Linux (`$IFS`) también puede funcionar ya que su valor predeterminado incluye un espacio y una pestaña, que funcionan como separadores entre argumentos de comando. Si usamos `${IFS}` donde deberían estar los espacios, la variable debería reemplazarse automáticamente con un espacio y nuestro comando debería funcionar.

Probemos `${IFS}` (`127.0.0.1%0a${IFS}`):

![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP configurada en '127.0.0.1%0a${IFS}'. La respuesta muestra HTML para un formulario de Host Checker y resultados de ping para 127.0.0.1.](<../../.gitbook/assets/cmdinj_filters_spaces_4 (1).jpg>)

Vemos que nuestra solicitud no fue denegada esta vez y volvimos a pasar por alto el filtro de espacio.
{% endstep %}

{% step %}
### Uso de la expansión de llaves (Brace Expansion)

Otra técnica es usar la característica de Bash llamada brace expansion (expansión de llaves), que añade automáticamente espacios entre argumentos envueltos entre llaves. Por ejemplo:

{% code title="Ejemplo de brace expansion" %}
```shell-session
OsmanMartinez@htb[/htb]$ {ls,-la}

total 0
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 13:01 ..
```
{% endcode %}

Como se ve, el comando se ejecutó correctamente sin usar espacios explícitos. Podemos aplicar lo mismo en omisiones de filtros de inyección de comandos, por ejemplo: `127.0.0.1%0a{ls,-la}`.

Para descubrir más desvíos de filtros espaciales, consulte la página de PayloadsAllTheThings sobre cómo escribir comandos sin espacios: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space
{% endstep %}
{% endstepper %}

{% hint style="info" %}
Ejercicio: Intente buscar otros métodos para eludir los filtros de espacio y utilícelos con la aplicación `Host Checker` para aprender cómo funcionan.
{% endhint %}
