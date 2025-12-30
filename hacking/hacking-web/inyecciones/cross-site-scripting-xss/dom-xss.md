# DOM XSS

El tercer y último tipo de XSS no persistente se denomina DOM-based XSS. Mientras que reflected XSS envía los datos de entrada al servidor back-end mediante solicitudes HTTP, el XSS DOM se procesa completamente en el lado del cliente mediante JavaScript. El XSS DOM se produce cuando se utiliza JavaScript para cambiar el código fuente de la página mediante el Document Object Model (DOM).

Podemos ejecutar el servidor a continuación para ver un ejemplo de una aplicación web vulnerable a XSS DOM. Podemos intentar añadir un elemento `test` y observamos que la aplicación web es similar a `To-Do List` las que usamos anteriormente:

![Lista de tareas con la opción "prueba" y el botón "Añadir". Próxima tarea: prueba.](../../.gitbook/assets/xss_dom_1.jpg)

Sin embargo, si abrimos la `Network` pestaña en las Herramientas para desarrolladores de Firefox y volvemos a agregar el `test` elemento, notaremos que no se realizan solicitudes HTTP:

![Instrucciones de la pestaña Red: Recargar para la actividad de la red, hacer clic en el cronómetro para el análisis del rendimiento.](../../.gitbook/assets/xss_dom_network.jpg)

Vemos que el parámetro de entrada en la URL usa un hashtag `#` para el elemento que añadimos, lo que significa que se trata de un parámetro del lado del cliente que se procesa completamente en el navegador. Esto indica que la entrada se procesa en el lado del cliente mediante JavaScript y nunca llega al backend; por lo tanto, es un `DOM-based XSS`.

Además, si consultamos el código fuente de la página haciendo clic en \[CTRL+U], observaremos que nuestra `test` cadena no se encuentra. Esto se debe a que el código JavaScript actualiza la página al hacer clic en el `Add` botón, lo cual ocurre después de que nuestro navegador haya recuperado el código fuente. Por lo tanto, el código fuente base no mostrará nuestra entrada, y si la actualizamos, no se conservará (es decir, `Non-Persistent`). Aún podemos ver el código fuente renderizado con la herramienta Web Inspector haciendo clic en \[CTRL+SHIFT+C]:

![Inspector HTML que muestra enlaces a Bootstrap y jQuery, con 'Siguiente tarea: prueba' en una lista.](../../.gitbook/assets/xss_dom_inspector.jpg)

***

### Fuente y sumidero

Para comprender mejor la naturaleza de la vulnerabilidad XSS basada en DOM, debemos comprender el concepto del `Source` y `Sink` mostrado en la página. `Source` es el objeto JavaScript que recibe la entrada del usuario; puede ser cualquier parámetro de entrada, como un parámetro de URL o un campo de entrada, como vimos anteriormente.

Por otro lado, `Sink` es la función que escribe la entrada del usuario en un objeto DOM de la página. Si la `Sink` función no depura correctamente la entrada del usuario, podría ser vulnerable a un ataque XSS. Algunas de las funciones de JavaScript más comunes para escribir en objetos DOM son:

* `document.write()`
* `DOM.innerHTML`
* `DOM.outerHTML`

Además, algunas de las funciones de la biblioteca jQuery que escriben en objetos DOM son:

* `add()`
* `after()`
* `append()`

Si una `Sink` función escribe la entrada exacta sin ninguna desinfección (como las funciones anteriores) y no se utilizaron otros medios de desinfección, entonces sabemos que la página debería ser vulnerable a XSS.

Podemos mirar el código fuente de la `To-Do` aplicación web y verificar `script.js`, y veremos que `Source` se toma del `task=` parámetro:

{% code title="script.js" %}
```javascript
var pos = document.URL.indexOf("task=");
var task = document.URL.substring(pos + 5, document.URL.length);
```
{% endcode %}

Justo debajo de estas líneas, vemos que la página utiliza la `innerHTML` función para escribir la `task` variable en el `todo` DOM:

{% code title="script.js" %}
```javascript
document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
```
{% endcode %}

Entonces, podemos ver que podemos controlar la entrada y la salida no se desinfecta, por lo que esta página debería ser vulnerable a DOM XSS.

***

### Ataques DOM

Si probamos la carga XSS que hemos usado anteriormente, veremos que no se ejecuta. Esto se debe a que la `innerHTML` función no permite el uso de `<script>` etiquetas como medida de seguridad. Aun así, existen muchas otras cargas XSS que usamos que no contienen `<script>` etiquetas, como la siguiente:

{% code title="payload.html" %}
```html
<img src="" onerror=alert(window.origin)>
```
{% endcode %}

La línea anterior crea un nuevo objeto de imagen HTML, que tiene un `onerror` atributo que puede ejecutar código JavaScript cuando no se encuentra la imagen. Por lo tanto, como proporcionamos un enlace de imagen vacío (`""`), nuestro código debería ejecutarse siempre sin necesidad de usar `<script>` etiquetas:

![Lista de tareas con entrada 'or=alert(window.origin)>' y botón "Añadir". Ventana emergente con IP 46.101.23.188:32648 y botón "Aceptar".](../../.gitbook/assets/xss_dom_alert.jpg)

Para atacar a un usuario con esta vulnerabilidad XSS del DOM, podemos copiar la URL del navegador y compartirla con él. Una vez que la visite, el código JavaScript debería ejecutarse. Ambas cargas útiles se encuentran entre las más básicas de XSS. En muchos casos, podríamos necesitar usar diferentes cargas útiles según la seguridad de la aplicación web y del navegador, lo cual analizaremos en la siguiente sección.
