---
icon: folder-magnifying-glass
---

# Unrestricted File Upload

Las vulnerabilidades de carga de archivos ocurren cuando una aplicación web no revisa bien qué archivos se suben.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
El `Content-Type` El encabezado de respuesta puede proporcionar pistas sobre qué tipo de archivo cree el servidor que ha servido. Si este encabezado no ha sido establecido explícitamente por el código de la aplicación, normalmente contiene el resultado del mapeo de extensión de archivo/tipo MIME.
{% endhint %}

El impacto de las vulnerabilidades de carga de archivos generalmente depende de dos factores clave:

* Qué aspecto del archivo el sitio web no logra validar correctamente, ya sea su tamaño, tipo, contenido, etc.
* Qué restricciones se imponen al archivo una vez que se ha cargado correctamente.

Aunque los desarrolladores suelen validar archivos, **si esas validaciones son débiles se pueden evadir**.&#x20;

Incluso si solo se permite subir ciertos tipos de archivos, aún pueden darse ataques como:

* XSS o XXE
* Denegación de servicio (DoS)
* Sobrescritura de archivos o configuraciones importantes



```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```

```php
<?php echo system($_GET['command']); ?>
```

{% hint style="info" %}
El `Content-Type` El encabezado de respuesta puede proporcionar pistas sobre qué tipo de archivo cree el servidor que ha servido. Si este encabezado no ha sido establecido explícitamente por el código de la aplicación, normalmente contiene el resultado del mapeo de extensión de archivo/tipo MIME.
{% endhint %}

## Validación defectuosa de las cargas de archivos

Cuando un formulario HTML envía datos simples (texto), el navegador usa:

* application/x-www-form-urlencoded

Este formato no es adecuado para subir archivos binarios (imágenes, PDFs, etc.). Para la subida de archivos se utiliza:

* multipart/form-data

#### ¿Qué es multipart/form-data?

Este tipo de petición divide el cuerpo del mensaje en partes independientes, una por cada campo del formulario.\
Cada parte puede incluir sus propios encabezados.

Ejemplo simplificado:

```html
POST /images HTTP/1.1
Host: normal-website.com
Content-Type: multipart/form-data; boundary=----boundary123
----boundary123
Content-Disposition: form-data; name="image"; filename="example.jpg"
Content-Type: image/jpeg
[binary content]
----boundary123--
```

#### Campos importantes en una subida de archivo

Cada parte del formulario suele incluir:

* Content-Disposition
  * Nombre del campo del formulario
  * Nombre del archivo (filename)
* Content-Type
  * Tipo MIME declarado por el cliente
  * Ejemplos: image/jpeg, image/png, application/pdf

#### Origen de la vulnerabilidad

Algunas aplicaciones validan los archivos únicamente comprobando:

* Content-Type

Ejemplo de validación defectuosa:

* El servidor solo permite image/jpeg o image/png
* Confía en el valor enviado por el navegador

Problemas:

* El cliente controla la petición
* El Content-Type puede modificarse
* No se valida el contenido real del archivo

#### Bypass típico

Un atacante puede:

* Subir un archivo que no es una imagen
* Cambiar el encabezado Content-Type a image/jpeg
* Enviar la petición manualmente (por ejemplo, con Burp Repeater)

Si el servidor confía solo en el MIME, el archivo será aceptado.
