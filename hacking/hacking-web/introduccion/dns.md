# DNS

El DNS (Domain Name System) se encarga de traducir nombres de dominio legibles a direcciones IP para que los equipos puedan comunicarse en red.

### Flujo de resolución DNS



{% stepper %}
{% step %}
### Consulta Local

* El sistema revisa primero la caché DNS y el archivo `hosts`.
* Si el dominio existe, se usa esa IP directamente.
{% endstep %}

{% step %}
### Consulta al resolver DNS

* Si no hay coincidencia local, se envía la consulta al resolver DNS (ISP o DNS público).
{% endstep %}

{% step %}
### Servidor raíz

* El resolver pregunta a un servidor raíz.
* Este indica qué servidor TLD debe consultarse.
{% endstep %}

{% step %}
### Servidor TLD

* Este servidor (por ejemplo <kbd>.com</kbd>) indica el servidor autoritativo del dominio.
{% endstep %}

{% step %}
### Servidor autoritativo

* Devuelve la dirección IP real del dominio solicitado.
{% endstep %}

{% step %}
### Respuesta final

Se entrega la IP al cliente y la  guarda en el caché.
{% endstep %}
{% endstepper %}

### Archivo hosts

Permite definir manualmente la relación entre dominio e IP, sin usar DNS externo.

**Ubicación:**

* Linux / macOS:

```
/etc/hosts
```

* Windows

```
C:\Windows\System32\drivers\etc\hosts
```

### Lista de Conceptos

| Concepto de DNS             | Descripción                                                                                  | Ejemplo                                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `Domain Name`               | Una etiqueta legible por humanos para un sitio web u otro recurso de Internet.               | `www.example.com`                                                                                                                        |
| `IP Address`                | Un identificador numérico único asignado a cada dispositivo conectado a Internet.            | `192.0.2.1`                                                                                                                              |
| `DNS Resolver`              | Un servidor que traduce nombres de dominio en direcciones IP.                                | El servidor DNS de su ISP o solucionadores públicos como Google DNS (`8.8.8.8`)                                                          |
| `Root Name Server`          | Los servidores de nivel superior en la jerarquía DNS.                                        | Hay 13 servidores raíz en todo el mundo, llamados AM: `a.root-servers.net`                                                               |
| `TLD Name Server`           | Servidores responsables de dominios de nivel superior específicos (por ejemplo, .com, .org). | [Verisign](https://en.wikipedia.org/wiki/Verisign) para `.com`, [PIR](https://en.wikipedia.org/wiki/Public_Interest_Registry) para`.org` |
| `Authoritative Name Server` | El servidor que contiene la dirección IP real de un dominio.                                 | A menudo administrado por proveedores de alojamiento o registradores de dominios.                                                        |
| `DNS Record Types`          | Diferentes tipos de información almacenadas en DNS.                                          | A, AAAA, CNAME, MX, NS, TXT, etc.                                                                                                        |

### Tipos de registros DNS

| Tipo de registro | Nombre completo                   | Descripción                                                                                                                                                              | Ejemplo de archivo de zona                                                                     |
| ---------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| `A`              | Registro de direcciones           | Asigna un nombre de host a su dirección IPv4.                                                                                                                            | `www.example.com.` EN A `192.0.2.1`                                                            |
| `AAAA`           | Registro de direcciones IPv6      | Asigna un nombre de host a su dirección IPv6.                                                                                                                            | `www.example.com.` EN AAAA `2001:db8:85a3::8a2e:370:7334`                                      |
| `CNAME`          | Registro de nombre canónico       | Crea un alias para un nombre de host, apuntándolo a otro nombre de host.                                                                                                 | `blog.example.com.` EN CNAME `webserver.example.net.`                                          |
| `MX`             | Registro de intercambio de correo | Especifica el(los) servidor(es) de correo responsables de manejar el correo electrónico para el dominio.                                                                 | `example.com.` EN MX 10 `mail.example.com.`                                                    |
| `NS`             | Registro del servidor de nombres  | Delega una zona DNS a un servidor de nombres autorizado específico.                                                                                                      | `example.com.` EN NS `ns1.example.com.`                                                        |
| `TXT`            | Registro de texto                 | Almacena información de texto arbitraria, a menudo utilizada para verificación de dominio o políticas de seguridad.                                                      | `example.com.` EN TXT `"v=spf1 mx -all"` (registro SPF)                                        |
| `SOA`            | Inicio del registro de autoridad  | Especifica información administrativa sobre una zona DNS, incluido el servidor de nombres principal, el correo electrónico de la persona responsable y otros parámetros. | `example.com.` EN SOA `ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400` |
| `SRV`            | Registro de servicio              | Define el nombre de host y el número de puerto para servicios específicos.                                                                                               | `_sip._udp.example.com.` EN SRV 10 5 5060 `sipserver.example.com.`                             |
| `PTR`            | Registro de puntero               | Se utiliza para búsquedas DNS inversas, asignando una dirección IP a un nombre de host.                                                                                  | `1.2.0.192.in-addr.arpa.` EN PTR `www.example.com.`                                            |
