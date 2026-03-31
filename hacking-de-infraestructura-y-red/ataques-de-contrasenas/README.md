---
icon: building-magnifying-glass
---

# Ataques de contraseñas

La tríada CIA (Confidentiality, Integrity, Availability) forma la base de toda estrategia de seguridad. Mantener ese equilibrio requiere auditar objetos y hosts, verificar que los usuarios tengan los permisos correctos (autorización) y confirmar su identidad antes de conceder acceso (autenticación). La mayoría de brechas de seguridad se remontan a la rotura de alguno de estos tres principios, y en particular al compromiso del mecanismo de autenticación.

### Factores de autenticación

La autenticación valida la identidad presentando uno o más factores a un mecanismo de validación:

| Factor                 | Descripción        | Ejemplo                                      |
| ---------------------- | ------------------ | -------------------------------------------- |
| **Something you know** | Secreto memorizado | Contraseña, PIN, frase de contraseña         |
| **Something you have** | Objeto físico      | Tarjeta inteligente, token OTP, teléfono     |
| **Something you are**  | Biometría          | Huella dactilar, reconocimiento facial, iris |
| **Somewhere you are**  | Geolocalización    | Dirección IP, país de origen                 |

El nivel de protección exigido determina cuántos factores se combinan. Entornos sanitarios o gubernamentales suelen requerir al menos dos (p. ej., CAC + PIN), mientras que servicios de consumo se conforman con usuario y contraseña más 2FA opcional.

### La contraseña como vector de ataque

A pesar de la proliferación de MFA, la contraseña sigue siendo el identificador de autenticación más extendido. Una contraseña de 8 caracteres usando solo mayúsculas y dígitos tiene 36⁸ ≈ 208 mil millones de combinaciones posibles, pero la realidad del comportamiento humano reduce drásticamente ese espacio efectivo:

* `123456` aparece más de 4,5 millones de veces en bases de datos de brechas filtradas.
* El 23% de los usuarios reutiliza contraseñas en tres o más cuentas.
* El 55% de los usuarios no cambia su contraseña ni después de ser notificados de una brecha.

> La reutilización de contraseñas es el multiplicador de daño más peligroso. Comprometer una sola cuenta puede dar acceso a docenas de servicios.

### Autorización vs. autenticación

Es fundamental no confundir los dos conceptos. La **autenticación** confirma _quién eres_; la **autorización** define _qué puedes hacer_. Una contraseña correcta autentica al usuario, pero los permisos asociados a esa cuenta determinan qué recursos puede acceder. Atacar contraseñas resuelve el primer problema; el escalado de privilegios resuelve el segundo.

### Herramientas para comprobar exposición

`HaveIBeenPwned` (haveibeenpwned.com) permite verificar si una dirección de correo electrónico aparece en bases de datos de brechas públicas. Es un recurso válido tanto para usuarios finales como para la fase de reconocimiento OSINT en un engagement autorizado.

> Durante reconocimiento, combinar HaveIBeenPwned con herramientas como `dehashed` o `LeakLookup` puede revelar contraseñas antiguas reutilizadas que siguen siendo válidas.
