# XSS reflejado

Existen dos tipos de vulnerabilidades Non-Persistent XSS: Reflected XSS, que es procesada por el servidor back-end, y DOM-based XSS, que se procesa completamente en el cliente y nunca llega al servidor back-end. A diferencia del XSS persistente, las vulnerabilidades Non-Persistent son temporales y no persisten tras las actualizaciones de la página. Por lo tanto, nuestros ataques solo afectan al usuario objetivo y no a otros usuarios que visiten la página.

Reflected XSS ocurre cuando nuestra entrada llega al servidor backend y se nos devuelve sin filtrar ni desinfectar. Hay muchos casos en los que la entrada completa podría devolverse, como mensajes de error o de confirmación. En estos casos, podemos intentar usar cargas útiles XSS para comprobar si se ejecutan. Sin embargo, como suelen ser mensajes temporales, una vez que abandonamos la página, no se ejecutarán de nuevo, por lo que son Non-Persistent.

Podemos iniciar el servidor a continuación para practicar en una página web vulnerable a una vulnerabilidad XSS reflejada. Es una To-Do List aplicación similar a la que usamos en la sección anterior. Podemos intentar agregar cualquier test cadena para ver cómo se gestiona:

![Interfaz de lista de tareas con una entrada de texto denominada "Tu tarea" y un botón "Añadir". Mensaje de error: "No se pudo añadir la tarea 'prueba'".](../../.gitbook/assets/xss_reflected_1.jpg)

Como podemos ver, obtenemos Task 'test' could not be added., que incluye nuestra entrada test como parte del mensaje de error. Si nuestra entrada no se filtró ni depuró, la página podría ser vulnerable a XSS. Podemos probar la misma carga útil XSS que usamos en la sección anterior y hacer clic en Add.

![Entrada de lista de tareas pendientes con etiqueta de script 'alert(window.origin)\</script>' y botón 'Agregar'.](../../.gitbook/assets/xss_reflected_2.jpg)

Una vez que hacemos clic Add, aparece la alerta emergente:

![Dirección IP 139.59.166.56:31323 con el botón 'Aceptar'.](../../.gitbook/assets/xss_stored_xss_alert.jpg)

En este caso, vemos que el mensaje de error ahora dice Task '' could not be added. Dado que nuestra carga útil está encapsulada en una etiqueta, el navegador no la procesa, por lo que aparecen comillas simples vacías ''. Podemos volver a consultar el código fuente de la página para confirmar que el mensaje de error incluye nuestra carga útil XSS:

Código: html

```html
<div></div><ul class="list-unstyled" id="todo"><div style="padding-left:25px">Task '<script>alert(window.origin)</script>' could not be added.</div></ul>
```

Como podemos ver, las comillas simples efectivamente contienen nuestra carga útil XSS 'alert(window.origin)'.

Si volvemos a visitar la Reflected página, el mensaje de error ya no aparece y nuestra carga XSS no se ejecuta, lo que significa que esta vulnerabilidad XSS es efectivamente Non-Persistent.

<details>

<summary>But if the XSS vulnerability is Non-Persistent, how would we target victims with it?</summary>

Esto depende de la solicitud HTTP utilizada para enviar nuestra entrada al servidor. Podemos comprobarlo en Firefox Developer Tools haciendo clic en \[CTRL+Shift+I] y seleccionando la pestaña Network. Después, podemos volver a introducir nuestra carga útil y hacer clic Add para enviarla:

![Pestaña de red que muestra solicitudes HTTP: estado 200 para index.php, bootstrap.min.js, jquery.min.js del host local; estado 404 para favicon.ico del host local.](../../.gitbook/assets/xss_reflected_network.jpg)

Como podemos ver, la primera fila muestra que nuestra solicitud fue una GET solicitud. GET envía sus parámetros y datos como parte de la URL. Por lo tanto, to target a user, we can send them a URL containing our payload. Para obtener la URL, podemos copiarla de la barra de URL en Firefox después de enviar nuestra carga XSS, o podemos hacer clic derecho en la GET solicitud en la Network pestaña y seleccionar Copy > Copy URL. Una vez que la víctima visita esta URL, la carga XSS se ejecutará:

![Dirección IP 139.59.166.56:31323 con el botón 'Aceptar'.](../../.gitbook/assets/xss_stored_xss_alert.jpg)

</details>
