# Cross-Site Scripting

Las vulnerabilidades de carga de archivos ocurren cuando una aplicación web no revisa bien qué archivos se suben.\
La más peligrosa es cuando **cualquier persona, sin iniciar sesión, puede subir cualquier archivo**, lo que puede permitir ejecutar código en el servidor.

Aunque los desarrolladores suelen validar archivos, **si esas validaciones son débiles se pueden evadir**.\
El ataque más grave consiste en subir un archivo malicioso (como un _web shell_) para **ejecutar comandos en el servidor** y tomar control del sistema.

Incluso si solo se permite subir ciertos tipos de archivos, aún pueden darse ataques como:

* XSS o XXE
* Denegación de servicio (DoS)
* Sobrescritura de archivos o configuraciones importantes

Estas vulnerabilidades también pueden aparecer por **usar librerías antiguas o inseguras**.\
Por eso, es clave aplicar **buenas prácticas de seguridad** para validar y manejar correctamente los archivos subidos.
