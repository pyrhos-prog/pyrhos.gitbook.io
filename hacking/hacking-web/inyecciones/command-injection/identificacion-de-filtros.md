# Identificacion de Filtros

\#commandInjection #comandos #injection

### Detección de filtros/WAF

Comencemos visitando la aplicación web en el ejercicio al final de esta sección. Vemos la misma aplicación web que hemos estado explotando, pero ahora tiene algunas mitigaciones bajo la manga. Podemos ver que si probamos los operadores anteriores que probamos, como (, , ), obtenemos el mensaje de error: `Host Checker``;``&&``||``invalid input`

![Filter](<../../.gitbook/assets/cmdinj_filters_1 (1).jpg>)

Esto indica que algo que enviamos activó un mecanismo de seguridad que denegó nuestra solicitud. Este mensaje de error se puede mostrar de varias maneras. En este caso, lo vemos en el campo donde se muestra la salida, lo que significa que fue detectado y evitado por la propia aplicación web. Si el mensaje de error mostrara una página diferente, con información como nuestra IP y nuestra petición, esto podría indicar que fue denegado por un WAF.

Comprobemos la carga útil que enviamos:

```bash
127.0.0.1; whoami
```

Aparte de la IP (que sabemos que no está en la lista negra), enviamos:

* Un carácter de punto y coma `;`
* Un espacio
* Un comando `whoami`

Por lo tanto, la aplicación web o el WAF, o ambas, detectaron uno o varios elementos. Veamos cómo omitir cada uno. Los mensajes que podríamos ver incluyen `detected a blacklisted character` o `detected a blacklisted command`.

***

### Personajes en la lista negra

Una aplicación web puede tener una lista de caracteres en la lista negra y, si el comando los contiene, denegaría la solicitud. El código puede tener un aspecto similar al siguiente:

```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

Si algún carácter de la cadena que enviamos coincide con un carácter de la lista negra, nuestra solicitud es denegada. Antes de comenzar nuestros intentos de eludir el filtro, debemos tratar de identificar qué carácter causó la solicitud denegada.

***

### Identificación de personajes en la lista negra

Reducimos nuestra solicitud a un carácter a la vez y vemos cuándo se bloquea. Sabemos que la carga útil básica funciona, así que comenzamos agregando el punto y coma (`;`):

```
127.0.0.1
127.0.0.1;
```

![Filter Character](<../../.gitbook/assets/cmdinj_filters_2 (1).jpg>)

Todavía recibimos un error `invalid input`, lo que significa que un punto y coma está en la lista negra. Entonces, veamos si todos los operadores de inyección que discutimos anteriormente están en la lista negra.
