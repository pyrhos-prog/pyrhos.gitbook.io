# Introducción a las inyecciones de comandos

\#command-injection #operadores #bugbounty #owasp-top-10

Una vulnerabilidad de inyección de comandos es uno de los tipos de vulnerabilidades más críticos. Nos permite ejecutar comandos del sistema directamente en el servidor de alojamiento back-end, lo que podría comprometer toda la red. Si una aplicación web utiliza una entrada controlada por el usuario para ejecutar un comando del sistema en el servidor back-end para recuperar y devolver una salida específica, es posible que podamos inyectar una carga útil maliciosa para subvertir el comando previsto y ejecutar nuestros comandos.

### ¿qué son las inyecciones

Las vulnerabilidades de inyección se consideran el riesgo número 3 en [Los 10 principales riesgos de las aplicaciones web según OWASP](https://owasp.org/www-project-top-ten/), dado su alto impacto y lo comunes que son. La inyección ocurre cuando la entrada controlada por el usuario se malinterpreta como parte de la consulta web o del código que se está ejecutando, lo que puede llevar a subvertir el resultado previsto de la consulta a un resultado diferente que sea útil para el atacante.

Hay muchos tipos de inyecciones que se encuentran en las aplicaciones web, dependiendo del tipo de consulta web que se ejecute. Los siguientes son algunos de los tipos de inyecciones más comunes:

| Inyección                                               | Descripción                                                                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Inyección de comandos del sistema operativo             | Ocurre cuando la entrada del usuario se utiliza directamente como parte de un comando del sistema operativo. |
| Inyección de código                                     | Ocurre cuando la entrada del usuario está directamente dentro de una función que evalúa el código.           |
| Inyecciones SQL                                         | Ocurre cuando la entrada del usuario se utiliza directamente como parte de una consulta SQL.                 |
| Inyección de HTML y secuencias de comandos entre sitios | Ocurre cuando se muestra la entrada exacta del usuario en una página web.                                    |

Existen muchos otros tipos de inyecciones además de las anteriores, como `LDAP injection`, `NoSQL Injection`, `HTTP Header Injection`, `XPath Injection`, `IMAP Injection`, `ORM Injection`, y otros. Siempre que se utiliza la entrada del usuario dentro de una consulta sin desinfectarla adecuadamente, puede ser posible escapar de los límites de la cadena de entrada del usuario a la consulta principal y manipularla para cambiar su propósito previsto. Es por esto que a medida que se introduzcan más tecnologías web en las aplicaciones web, veremos nuevos tipos de inyecciones introducidas en las aplicaciones web.

### Inyecciones de comandos del sistema operativo

Cuando se trata de inyecciones de comandos del sistema operativo, la entrada del usuario que controlamos debe ingresar directa o indirectamente (o afectar de alguna manera) a una consulta web que ejecuta comandos del sistema. Todos los lenguajes de programación web tienen diferentes funciones que permiten al desarrollador ejecutar comandos del sistema operativo directamente en el servidor back-end cuando lo necesite. Esto se puede utilizar para diversos fines, como instalar complementos o ejecutar determinadas aplicaciones.

**Ejemplo de PHP**

Por ejemplo, una aplicación web escrita en `PHP` puede utilizar el `exec`, `system`, `shell_exec`, `passthru`, o `popen` funciones para ejecutar comandos directamente en el servidor back-end, cada una con un caso de uso ligeramente diferente. El siguiente código es un ejemplo de código PHP que es vulnerable a inyecciones de comandos:

{% code title="vulnerable.php" %}
```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```
{% endcode %}

Quizás una aplicación web en particular tenga una funcionalidad que permita a los usuarios crear una nueva `.pdf` documento que se crea en el `/tmp` directorio con un nombre de archivo proporcionado por el usuario y luego puede ser utilizado por la aplicación web para fines de procesamiento de documentos. Sin embargo, como la entrada del usuario desde el `filename` parámetro en el `GET` La solicitud se utiliza directamente con el `touch` comando (sin ser desinfectado ni escapado primero), la aplicación web se vuelve vulnerable a la inyección de comandos del sistema operativo. Esta falla se puede aprovechar para ejecutar comandos arbitrarios del sistema en el servidor back-end.

**Ejemplo de NodeJS**

Esto no es exclusivo de `PHP` solamente, pero puede ocurrir en cualquier marco o lenguaje de desarrollo web. Por ejemplo, si se desarrolla una aplicación web en `NodeJS`, un desarrollador puede usarlo `child_process.exec` o `child_process.spawn` para el mismo propósito. El siguiente ejemplo realiza una funcionalidad similar a la que analizamos anteriormente:

{% code title="vulnerable.js" %}
```javascript
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```
{% endcode %}

El código anterior también es vulnerable a una vulnerabilidad de inyección de comandos, ya que utiliza el `filename` parámetro del `GET` solicitar como parte del comando sin desinfectarlo primero. Ambos `PHP` y `NodeJS` Las aplicaciones web se pueden explotar utilizando los mismos métodos de inyección de comandos.

Asimismo, otros lenguajes de programación de desarrollo web tienen funciones similares utilizadas para los mismos fines y, si son vulnerables, pueden explotarse utilizando los mismos métodos de inyección de comandos. Además, las vulnerabilidades de inyección de comandos no son exclusivas de las aplicaciones web, sino que también pueden afectar a otros binarios y clientes gruesos si pasan una entrada de usuario no desinfectada a una función que ejecuta comandos del sistema, lo que también puede explotarse con los mismos métodos de inyección de comandos.

La siguiente sección analizará diferentes métodos para detectar y explotar vulnerabilidades de inyección de comandos en aplicaciones web.
