# Indentificacion de filtros

Como hemos visto en la sección anterior, incluso si los desarrolladores intentan proteger la aplicación web contra inyecciones, aún puede ser explotable si no está codificada de forma segura. Otro tipo de mitigación de inyección es utilizar caracteres y palabras incluidos en la lista negra del back-end para detectar intentos de inyección y rechazar la solicitud si alguna solicitud los contenía. Otra capa adicional a esto es el uso de firewalls de aplicaciones web (WAF), que pueden tener un alcance más amplio y varios métodos de detección de inyecciones y prevenir otros ataques como inyecciones SQL o ataques XSS.

En esta sección veremos algunos ejemplos de cómo se pueden detectar y bloquear las inyecciones de comandos y cómo podemos identificar qué se está bloqueando.

***

### Detección de filtros/WAF

Comencemos visitando la aplicación web en el ejercicio al final de esta sección. Vemos lo mismo `Host Checker` aplicación web que hemos estado explotando, pero ahora tiene algunas mitigaciones bajo la manga. Podemos ver que si probamos los operadores anteriores que probamos, como (`;`, `&&`, `||`), recibimos el mensaje de error `invalid input`:\
![Interface showing an HTTP request and response. The request includes headers like Host and User-Agent, with IP set to '127.0.0.1+%3b+whoami'. The response displays a Host Checker form with '127.0.0.1' entered and an 'Invalid input' message.](<../../.gitbook/assets/cmdinj_filters_1 (1).jpg>)

Esto indica que algo que enviamos activó un mecanismo de seguridad que denegó nuestra solicitud. Este mensaje de error se puede mostrar de varias maneras. En este caso, lo vemos en el campo donde se muestra la salida, lo que significa que fue detectada y prevenida por el `PHP` aplicación web en sí. If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF.

Comprobemos la carga útil que enviamos:

{% code title="payload.sh" %}
```bash
127.0.0.1; whoami
```
{% endcode %}

Aparte de la IP (que sabemos que no está en la lista negra), enviamos:

{% stepper %}
{% step %}
### Un punto y coma

`;`
{% endstep %}

{% step %}
### Un espacio

Un carácter de espacio.
{% endstep %}

{% step %}
### Un comando

`whoami`
{% endstep %}
{% endstepper %}

So, the web application either `detected a blacklisted character` or `detected a blacklisted command`, or both. So, let us see how to bypass each.

***

### Blacklisted Characters

A web application may have a list of blacklisted characters, and if the command contains them, it would deny the request. The `PHP` code may look something like the following:

{% code title="filter.php" %}
```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```
{% endcode %}

If any character in the string we sent matches a character in the blacklist, our request is denied. Before we start our attempts at bypassing the filter, we should try to identify which character caused the denied request.

***

### Identifying Blacklisted Character

Let us reduce our request to one character at a time and see when it gets blocked. We know that the (`127.0.0.1`) payload does work, so let us start by adding the semi-colon (`127.0.0.1;`):\
![Interface showing an HTTP request and response. The request includes headers like Host and User-Agent, with IP set to '127.0.0.1%3b'. The response displays HTML for a Host Checker form with an 'Invalid input' message.](<../../.gitbook/assets/cmdinj_filters_2 (1).jpg>)

We still get an `invalid input` error meaning that a semi-colon is blacklisted. So, let's see if all of the injection operators we discussed previously are blacklisted.
