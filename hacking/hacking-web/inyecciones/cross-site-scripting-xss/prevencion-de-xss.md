# Prevencion de XSS

A estas alturas, deberíamos tener una buena comprensión de qué es una vulnerabilidad XSS y sus diferentes tipos, cómo detectarla y cómo explotarla. Concluiremos el módulo aprendiendo a defendernos de las vulnerabilidades XSS.

Como se mencionó anteriormente, las vulnerabilidades XSS se relacionan principalmente con dos partes de la aplicación web: una, `Source` como un campo de entrada de usuario, y otra `Sink`, que muestra los datos de entrada. Estos son los dos puntos principales que debemos asegurar, tanto en el front-end como en el back-end.

El aspecto más importante para prevenir vulnerabilidades XSS es la correcta limpieza y validación de la entrada, tanto en el front-end como en el back-end. Además, se pueden tomar otras medidas de seguridad para prevenir ataques XSS.

***

### Interfaz

Como el front-end de la aplicación web es de donde se toma la mayor parte (pero no toda) de la entrada del usuario, es esencial depurar y validar la entrada del usuario en el front-end usando JavaScript.

**Validación de entrada**

Por ejemplo, en el ejercicio de esta `XSS Discovery` sección, vimos que la aplicación web no permite enviar el formulario si el formato del correo electrónico no es válido. Esto se solucionó con el siguiente código JavaScript:

{% code title="validateEmail (JavaScript)" %}
```javascript
function validateEmail(email) {
    const re = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
    return re.test($("#login input[name=email]").val());
}
```
{% endcode %}

Como podemos ver, este código prueba el `email` campo de entrada y devuelve `true` si coincide con la validación Regex de un formato de correo electrónico, y `false` en caso contrario.

**Sanitización de entrada**

Además de la validación de entrada, debemos asegurarnos siempre de no permitir ninguna entrada con código JavaScript, escapando cualquier carácter especial. Para ello, podemos utilizar la biblioteca de JavaScript DOMPurify, como se indica a continuación: https://github.com/cure53/DOMPurify

{% code title="Uso de DOMPurify (JavaScript)" %}
```javascript
<script type="text/javascript" src="dist/purify.min.js"></script>
let clean = DOMPurify.sanitize( dirty );
```
{% endcode %}

Esto escapará cualquier carácter especial con una barra invertida `\`, lo que debería ayudar a garantizar que un usuario no envíe ninguna entrada con caracteres especiales (como código JavaScript), lo que debería evitar vulnerabilidades como DOM XSS.

**Entrada directa**

Nunca debemos introducir la entrada del usuario directamente dentro de ciertas partes del HTML. Si la entrada del usuario se introduce en cualquiera de los ejemplos siguientes, puede inyectar código JavaScript malicioso y provocar una vulnerabilidad XSS.

{% stepper %}
{% step %}
### EtiquetasNo insertar entrada de usuario dentro de bloques de código JavaScript: `<script></script>`.
{% endstep %}

{% step %}
### EtiquetasNo insertar entrada de usuario dentro de bloques de estilo CSS: `<style></style>`.
{% endstep %}

{% step %}
### Atributos/etiquetas

No usar la entrada del usuario directamente en atributos o nombres de etiqueta: `<div name='INPUT'></div>`.
{% endstep %}

{% step %}
### Comentarios HTML

No introducir entrada de usuario dentro de comentarios HTML: `<!-- -->`.
{% endstep %}
{% endstepper %}

Además, debemos evitar el uso de funciones de JavaScript que permitan modificar el HTML con texto sin escapar, por ejemplo:

* DOM.innerHTML
* DOM.outerHTML
* document.write()
* document.writeln()
* document.domain

Y las siguientes funciones jQuery:

* html()
* parseHTML()
* add()
* append()
* prepend()
* after()
* insertAfter()
* before()
* insertBefore()
* replaceAll()
* replaceWith()

Estas funciones escriben texto sin escapar en el HTML; si el usuario ingresa contenido en ellas, puede incluir código JavaScript malicioso y generar una vulnerabilidad XSS.

***

### Back-end

También debemos asegurarnos de prevenir vulnerabilidades XSS con medidas en el backend para prevenir vulnerabilidades XSS almacenadas y reflejadas. Como vimos en el `XSS Discovery` ejercicio de la sección, aunque contaba con validación de entrada en el frontend, esto no fue suficiente para evitar la inyección de una carga maliciosa en el formulario. Por lo tanto, también debemos implementar medidas de prevención XSS en el backend. Esto se puede lograr con la sanitización y validación de entrada y salida, la configuración del servidor y herramientas de backend que ayudan a prevenir vulnerabilidades XSS.

**Validación de entrada**

La validación de entrada en el backend es bastante similar a la del frontend y utiliza expresiones regulares o funciones de biblioteca para garantizar que el campo de entrada sea el esperado. Si no coincide, el servidor backend lo rechazará y no lo mostrará.

Un ejemplo de validación de correo electrónico en un back-end PHP es el siguiente:

{% code title="Validación de email (PHP)" %}
```php
if (filter_var($_GET['email'], FILTER_VALIDATE_EMAIL)) {
    // do task
} else {
    // reject input - do not display it
}
```
{% endcode %}

Para un back-end NodeJS, podemos usar el mismo código JavaScript mencionado anteriormente para el front-end.

**Sanitización de entrada**

En lo que respecta a la limpieza de entrada, el backend desempeña un papel fundamental, ya que la limpieza de entrada del frontend puede obviarse fácilmente mediante el envío de solicitudes personalizadas `GET` o `POST`. Afortunadamente, existen bibliotecas muy robustas para diversos lenguajes de backend que pueden limpiar adecuadamente cualquier entrada del usuario, garantizando así que no se produzcan inyecciones.

Por ejemplo, para un back-end PHP, podemos usar la función `addslashes` para desinfectar la entrada del usuario escapando caracteres especiales con una barra invertida:

{% code title="addslashes (PHP)" %}
```php
addslashes($_GET['email'])
```
{% endcode %}

En cualquier caso, la entrada directa del usuario (por ejemplo `$_GET['email']`) nunca debe mostrarse directamente en la página, ya que esto puede generar vulnerabilidades XSS.

Para un back-end NodeJS, también podemos usar la biblioteca DOMPurify como hicimos con el front-end, de la siguiente manera:

{% code title="DOMPurify en NodeJS (JavaScript)" %}
```javascript
import DOMPurify from 'dompurify';
var clean = DOMPurify.sanitize(dirty);
```
{% endcode %}

**Codificación HTML de salida**

Otro aspecto importante a tener en cuenta en el backend es `Output Encoding`. Esto significa que debemos codificar cualquier carácter especial en su entidad HTML, lo cual resulta útil si necesitamos mostrar toda la entrada del usuario sin introducir una vulnerabilidad XSS. En un backend PHP, podemos usar las funciones `htmlspecialchars` o `htmlentities`, que codificarían ciertos caracteres especiales en su entidad HTML (por ejemplo, `<` en `&lt;`), de modo que el navegador los muestre correctamente, pero sin causar ninguna inyección de código.

{% code title="htmlentities (PHP)" %}
```php
htmlentities($_GET['email']);
```
{% endcode %}

Para un back-end NodeJS, podemos usar cualquier biblioteca que realice codificación HTML, como `html-entities`, de la siguiente manera:

{% code title="html-entities (JavaScript)" %}
```javascript
import encode from 'html-entities';
encode('<'); // -> '&lt;'
```
{% endcode %}

Una vez que nos aseguremos de que toda la entrada del usuario esté validada, desinfectada y codificada en la salida, deberíamos reducir significativamente el riesgo de tener vulnerabilidades XSS.

**Configuración del servidor**

Además de lo anterior, existen ciertas configuraciones de servidor web back-end que pueden ayudar a prevenir ataques XSS, como:

* Usar HTTPS en todo el dominio.
* Uso de encabezados de prevención XSS.
* Utilizar el tipo de contenido apropiado para la página, por ejemplo `X-Content-Type-Options=nosniff`.
* Usar `Content-Security-Policy` con directivas como `script-src 'self'`, que sólo permite scripts alojados localmente.
* Usar los indicadores de cookies `HttpOnly` y `Secure` para evitar que JavaScript lea las cookies y solo las transporte a través de HTTPS.

Además de lo anterior, contar con un buen framework `Web Application Firewall (WAF)` puede reducir significativamente las posibilidades de explotación XSS, ya que detectará automáticamente cualquier tipo de inyección a través de solicitudes HTTP y las rechazará automáticamente. Además, algunos frameworks, como ASP.NET, ofrecen protección XSS integrada: https://learn.microsoft.com/en-us/aspnet/core/security/cross-site-scripting?view=aspnetcore-7.0

En definitiva, debemos hacer todo lo posible para proteger nuestras aplicaciones web contra vulnerabilidades XSS utilizando estas técnicas de prevención. Incluso después de todo esto, debemos practicar todas las habilidades aprendidas en este módulo e intentar identificar y explotar las vulnerabilidades XSS en cualquier campo de entrada potencial, ya que la codificación y las configuraciones seguras aún pueden dejar brechas y vulnerabilidades que puedan explotarse. Si practicamos la defensa del sitio web utilizando ambas técnicas `offensive` y `defensive`, alcanzaremos un nivel confiable de seguridad contra vulnerabilidades XSS.
