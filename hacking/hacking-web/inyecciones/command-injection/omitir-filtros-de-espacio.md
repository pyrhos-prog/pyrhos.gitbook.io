# Omitir filtros de espacio

\#commandInjection #comandos #injection

### Omitir operadores de la lista negra

Veremos que la mayoría de los operadores de inyección están en la lista negra. Sin embargo, el carácter de nueva línea generalmente no está en la lista negra, ya que puede ser necesario en la propia carga útil. Sabemos que el carácter de nueva línea funciona para agregar nuestros comandos tanto en Linux como en Windows, así que intentemos usarlo como nuestro operador de inyección:

![Operador de filtro](<../../.gitbook/assets/cmdinj_filters_operator (1).jpg>)

Como podemos ver, a pesar de que nuestra carga útil incluía un carácter de nueva línea, nuestra solicitud no fue denegada y obtuvimos la salida del comando ping. Comencemos discutiendo cómo omitir un carácter comúnmente incluido en la lista negra: un carácter de espacio.

***

### Omitir espacios en la lista negra

Ahora que tenemos un operador de inyección en funcionamiento, modifiquemos nuestra carga útil original y la enviemos de nuevo como:

```
127.0.0.1%0a whoami
```

![Espacio de filtro](<../../.gitbook/assets/cmdinj_filters_spaces_1 (1).jpg>)

Como podemos ver, todavía recibimos un mensaje de error, lo que significa que todavía tenemos otros filtros que omitir. Entonces, como hicimos antes, agreguemos solo el siguiente carácter (que es un espacio) y veamos si causó la solicitud denegada:

```
invalid input
```

![Espacio de filtro](<../../.gitbook/assets/cmdinj_filters_spaces_2 (1).jpg>)

Como podemos ver, el espacio también está en la lista negra. Un espacio es un carácter comúnmente incluido en la lista negra, especialmente si la entrada no debe contener espacios, como una IP, por ejemplo. Aún así, ¡hay muchas maneras de agregar un carácter espacial sin usar realmente el carácter espacial!

{% stepper %}
{% step %}
### Uso de pestañas

El uso de tabulaciones (%09) en lugar de espacios es una técnica que puede funcionar, ya que tanto Linux como Windows aceptan comandos con tabulaciones entre argumentos, y se ejecutan de la misma manera. Intentemos usar una tabulación en lugar del carácter de espacio:

```
127.0.0.1%0a%09
```

![Espacio de filtro](<../../.gitbook/assets/cmdinj_filters_spaces_3 (1).jpg>)

Como podemos ver, omitimos con éxito el filtro de caracteres de espacio mediante el uso de una tabulación en su lugar.
{% endstep %}

{% step %}
### Uso de $IFS

El uso de la variable de entorno de Linux ($IFS) también puede funcionar, ya que su valor predeterminado es un espacio y una tabulación, que funcionarían entre los argumentos de los comandos. Por lo tanto, si usamos ${IFS} donde deberían estar los espacios, la variable debería reemplazarse automáticamente con un espacio y nuestro comando debería funcionar.

Ejemplo de payload:

```
127.0.0.1%0a${IFS}
```

![Espacio de filtro](<../../.gitbook/assets/cmdinj_filters_spaces_4 (1).jpg>)

Vemos que nuestra solicitud no fue denegada esta vez, y volvimos a pasar por alto el filtro de espacio.
{% endstep %}

{% step %}
### Uso de la expansión de llaves (Brace Expansion)

Otra técnica es usar la expansión de llaves de Bash, que puede insertar automáticamente espacios entre argumentos envueltos entre llaves. Por ejemplo:

```shell-session
zunderrubb@htb[/htb]$ {ls,-la}

total 0
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 13:01 ..
```

El comando se ejecutó con éxito sin usar espacios. Podemos usar el mismo método en la omisión del filtro de inyección de comandos, por ejemplo:

```
127.0.0.1%0a{ls,-la}
```

En el laboratorio vemos cómo funciona el uso de omitir el espacio para encadenar no solo uno sino dos comandos diferentes.

!\[\[Pasted image 20240328221930.png]]
{% endstep %}
{% endstepper %}

{% hint style="info" %}
Para más técnicas de bypass sin espacios, consulta la colección de PayloadsAllTheThings:\
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space
{% endhint %}
