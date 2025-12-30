# Protocolo http

La mayoría de las comunicaciones por Internet se realizan con solicitudes web a través del protocolo HTTP.

&#x20;[HTTP](https://tools.ietf.org/html/rfc2616) es un protocolo de nivel de aplicación que se utiliza para acceder a los recursos de la World Wide Web. El término significa texto que contiene enlaces a otros recursos y texto que los lectores pueden interpretar fácilmente.

* La comunicación HTTP consta de un cliente y un servidor
* El cliente solicita al servidor un recurso.&#x20;
* El servidor procesa las solicitudes y devuelve el recurso solicitado.&#x20;
* El puerto predeterminado para la comunicación HTTP es el puerto `80`, aunque se puede cambiar a cualquier otro puerto, dependiendo de la configuración del servidor web.&#x20;
* Las mismas solicitudes se utilizan cuando usamos Internet para visitar diferentes sitios web.&#x20;
* Introducimos un Fully Qualified Domain Name (FQDN) como `www.hackthebox.com` (un Uniform Resource Locator — `URL`) para llegar al sitio web deseado.

### Flujo HTTP

![Flujo http](../.gitbook/assets/HTTP_Flow.png)

El diagrama anterior presenta la anatomía de una solicitud HTTP a alto nivel. A continuación se describen los pasos principales del flujo cuando un usuario introduce una URL en el navegador:

{% stepper %}
{% step %}
### Resolución DNS

* Cuando un usuario introduce la URL en el navegador, este envía una solicitud a un servidor DNS para resolver el dominio y obtener su IP.
* El servidor DNS busca la dirección IP y la devuelve.&#x20;
* Todos los nombres de dominio deben resolverse de esta manera, ya que un servidor no puede comunicarse sin una dirección IP.

{% hint style="info" %}
Nuestros navegadores suelen buscar primero los registros en el archivo local y si el dominio solicitado no existe se pondrían en contacto con otros servidores DNS.

&#x20;Podemos usar `/etc/hosts` para agregar registros manuales para la resolución de DNS, añadiendo la IP seguida del nombre de dominio.
{% endhint %}
{% endstep %}

{% step %}
### Solicitud HTTP al servidor

Una vez que el navegador obtiene la dirección IP envía una solicitud GET al puerto HTTP predeterminado (por ejemplo, `80`), solicitando la ruta raíz (`/`). El servidor web recibe la solicitud y la procesa. De forma predeterminada, los servidores están configurados para devolver un archivo de índice cuando se recibe una solicitud (por ejemplo, `index.html`).
{% endstep %}

{% step %}
### Respuesta HTTP

En este caso, el contenido de `index.html` es leído y devuelto como una respuesta HTTP. La respuesta también contiene el código de estado  `200 OK` que indica que la solicitud se ha procesado correctamente. A continuación, el navegador web procesa el contenido y lo presenta al usuario.
{% endstep %}
{% endstepper %}
