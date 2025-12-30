---
icon: syringe
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# OWASP TOP 10

El OWASP TOP 10 es un documento que recopila los 10 grupos de vulnerabilidades web mas importantes.

{% tabs %}
{% tab title="1. Broken Acces Authetication" %}
### Broken Acces Authentication

* IDOR (Insecure Direct Object References): Acceder a datos de otros usuarios cambiando un ID en la URL (ej. `ver-factura?id=123`).
* Path Traversal (Directory Traversal): Acceder a archivos del servidor fuera del directorio web (ej. `../../etc/passwd`).
* CSRF (Cross-Site Request Forgery): Forzar a un usuario logueado a realizar una acción que no desea (ej. cambiar su contraseña) sin que se dé cuenta.


{% endtab %}

{% tab title="2. Fallos Criptográficos" %}
* No usar HTTPS&#x20;
* Hashes débiles para almacenar datos como contraseñas
* Guardar crendenciales en texto plano
{% endtab %}

{% tab title="3. Inyecciones" %}
* SQL Injection (SQLi): Inyectar código SQL para robar, modificar o borrar la base de datos.
* Cross-Site Scripting (XSS): Inyectar código JavaScript que se ejecuta en el navegador de otra víctima para robar su sesión.
* SSTI (Server-Side Template Injection): Inyectar código en el motor de plantillas del servidor para ejecutar comandos.
* XXE (XML External Entity): Inyectar entidades XML maliciosas para leer archivos locales del servidor o atacar la red interna.
* Command Injection: Inyectar comandos del sistema operativo (Linux/Windows) para que el servidor los ejecute.
{% endtab %}

{% tab title="4. Diseño Inseguro" %}
* Unrestricted File Upload: Permitir subir archivos (como `.php` o `.jsp`) que pueden ejecutar código en el servidor.
* Falta de Límites de Peticiones: No limitar los intentos de login, permitiendo ataques de fuerza bruta.
{% endtab %}

{% tab title="5. Configuración de seguridad incorrecta" %}
* Credenciales por defecto: Dejar `admin:admin` en un panel, base de datos o dispositivo.
* Modo de depuración (Debug) activo: Mostrar errores detallados al público que revelan información interna.
* Cabeceras de seguridad faltantes: No configurar cabeceras HTTP (como CSP o HSTS) que protegen al navegador.
{% endtab %}

{% tab title="6. Componente vulnerables y desactualizados" %}
* Uso de librerías con CVEs: Utilizar software de terceros (ej. una librería de npm o Java) que tiene un fallo de seguridad conocido y público (CVE).
* Software sin parches: No aplicar las actualizaciones de seguridad al sistema operativo o al _framework_ (ej. un WordPress desactualizado).
{% endtab %}

{% tab title="7. Fallos de identificación" %}
* Credential Stuffing (Relleno de credenciales): Permitir probar miles de contraseñas robadas de otros sitios sin ningún bloqueo.
* Falta de MFA (Autenticación Multifactor): Proteger una cuenta solo con una contraseña, sin un segundo factor de verificación (como un código).
* Gestión de sesión insegura: Usar IDs de sesión que nunca caducan o que son fáciles de predecir.
{% endtab %}

{% tab title="8. Fallos de integridad del software" %}
* Deserialización Insegura: Confiar ciegamente en objetos de datos que envía un usuario, permitiendo que inyecte código malicioso.
* Actualización de software no verificada: Aceptar e instalar una actualización sin comprobar su firma digital (permitiendo un ataque a la cadena de suministro).


{% endtab %}

{% tab title="9. Fallos de registro y monitoreo" %}
* Registro insuficiente: No guardar logs de quién inicia sesión, desde dónde, o quién accede a datos sensibles.
{% endtab %}

{% tab title="10. Server-Side Request Forgery" %}
* SSRF (Server-Side Request Forgery): Engañar al servidor para que realice peticiones de red a sistemas internos (como una base de datos) o externos en su nombre.
{% endtab %}
{% endtabs %}
