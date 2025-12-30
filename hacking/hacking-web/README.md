# Funcionamiento de una web

Se accede a los recursos sobre HTTP a través de una `URL`, que ofrece muchas más especificaciones que simplemente especificar un sitio web que queremos visitar:

![estructura de una url](.gitbook/assets/url_structure.png)

Esto es lo que representa cada componente:

<table data-header-hidden data-full-width="false"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Componente</strong></td><td><strong>Ejemplo</strong></td><td><strong>Descripción</strong></td></tr><tr><td><code>Scheme</code></td><td><code>http://</code> <code>https://</code></td><td>Identifica el protocolo al que accede el cliente</td></tr><tr><td><code>User Info</code></td><td><code>admin:password@</code></td><td>Componente opcional que contiene las credenciales para autentica al host, se separa con <kbd>@</kbd></td></tr><tr><td><code>Host</code></td><td><code>inlanefreight.com</code></td><td>Es la ubicación del recurso, puede ser un dominio o una dirección IP.</td></tr><tr><td><code>Port</code></td><td><code>:80</code></td><td>El puerto esta separado por un colon (:), si no se especifica ningun puerto el sistema usa automaticamente el 80 (http) o el 443 (https).</td></tr><tr><td><code>Path</code></td><td><code>/dashboard.php</code></td><td>Apunta al recurso al que se esta accediendo, puede ser un archivo o una carpeta.</td></tr><tr><td><code>Query String</code></td><td><code>?login=true</code></td><td>La consulta empieza con un signo de interrogación, y consiste en un parámetro <code>login</code> y un valor <code>true</code>. Múltiples parámetros pueden separarse por un ampersand <code>&#x26;</code>.</td></tr><tr><td><code>Fragments</code></td><td><code>#status</code></td><td>Son procesados por los navegadores del lado del cliente para localizar secciones dentro del recurso primario (una cabecera o sección en la página).</td></tr></tbody></table>

