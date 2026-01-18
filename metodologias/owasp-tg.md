---
icon: arrow-progress
---

# OWASP TG

## OWASP Testing Guide (OWASP TG)

> OWASP Testing Guide (OWASP TG) es una metodología orientada a la evaluación de la seguridad en aplicaciones web, centrada en identificar fallos de diseño, implementación y configuración.



### Enfoque de OWASP TG

OWASP TG se centra en:

* Aplicaciones web
* APIs
* Servicios web
* Lógica de negocio
* Configuraciones inseguras

### Fases del OWASP Testing Guide

{% stepper %}
{% step %}
#### Information Gathering

* Identificación de la aplicación
* Enumeración de endpoints
* Descubrimiento de tecnologías
* Análisis de superficie de ataque
{% endstep %}

{% step %}
#### Configuration and Deployment Management Testing

* Configuraciones inseguras
* Archivos expuestos
* Credenciales por defecto
* Gestión de errores
{% endstep %}

{% step %}
#### Identity Management Testing

* Gestión de usuarios
* Creación y eliminación de cuentas
* Enumeración de usuarios
{% endstep %}

{% step %}
#### Authentication Testing

* Mecanismos de autenticación
* Fuerza de contraseñas
* Gestión de sesiones
{% endstep %}

{% step %}
#### Authorization Testing

* Control de accesos
* Escalada de privilegios
* Fallos de autorización
{% endstep %}

{% step %}
#### Session Management Testing

* Cookies
* Tokens
* Expiración de sesión
* Protección contra hijacking
{% endstep %}

{% step %}
#### Input Validation Testing

* Inyección SQL
* XSS
* Command Injection
* Validación de entradas
{% endstep %}

{% step %}
#### Error Handling and Logging

* Mensajes de error
* Fugas de información
* Registros de seguridad
{% endstep %}

{% step %}
#### Business Logic Testing

* Flujos de negocio
* Controles lógicos
* Bypass de restricciones
{% endstep %}

{% step %}
#### Client-Side Testing

* JavaScript
* DOM-based XSS
* Manipulación del cliente
{% endstep %}
{% endstepper %}

### Uso habitual

OWASP Testing Guide se utiliza en:

* Pentesting web
* Auditorías de aplicaciones
* Desarrollo seguro (SDLC)
* Revisión de APIs
* Formación en AppSec
