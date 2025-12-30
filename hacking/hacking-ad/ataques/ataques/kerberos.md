# Kerberos

* TGT -> Ticket Granting Ticket: Que es el ticket que es presentado al KDC para obtener el TGS
* TGS -> Ticket Granting Service: Es el ticket que es presentado a cuyos recursos nos queremos conectar.
* KDC -> Key Distribution Center: Es el servicio de kerberos encargado de distribuir los tickets a los usuarios normalmente esta instalado en el DC
* AP -> Application Server: Servicio que ofrecen los recursos a los que nos que nos queremos conectar

{% stepper %}
{% step %}
### Flujo normal de Kerberos — AS (Autenticación)

* El usuario introduce la contraseña, y esta se convierte en un hash NTLM que se envía al KDC.
* A este proceso se le llama AS-REQ.
* Se está solicitando un TGT al KDC.
* Tras validar, el KDC devuelve un TGT cifrado y firmado: AS-REP.
{% endstep %}

{% step %}
### Flujo normal de Kerberos — TGS (Delegación de tickets)

* Con el TGT obtenido, el usuario solicita un TGS para el servicio destino.
* Esta petición se cifra con el hash NTLM del servicio: TGS-REQ.
* Tras validaciones, el KDC devuelve un TGS: TGS-REP.
{% endstep %}

{% step %}
### Flujo normal de Kerberos — AP (Acceso al servicio)

* Con el TGS se hace la petición al Application Server (AP): AP-REQ.
* Si todo es correcto, el AP responde con AP-REP.
{% endstep %}
{% endstepper %}

!\[\[Captura de pantalla 2024-05-31 025821.png]]

***

* Gracias a este flujo se pueden realizar ataques como:
  * `Silver Ticket Attack` -> Pretende enviar un TGS al servidor para que este nos responda y gracias a eso poder llegar a acceder a los recursos del sistema.
  * `Golden Ticket Attack` -> Se centra en generar un TGT que se presenta al KDC para obtener un TGS.
* Para generar un TGS necesitaremos 3 valores:
  1. Un hash NTLM del servicio correspondiente
  2. Domain SID (getPac.py)
  3. El SPN

Con estos tres valores podemos tratar de generar un TGS. Este TGS puedes generarlo desde el usuario que nosotros queramos, para tratar de conectarte directamente al AP y tratar de conseguir directamente un AP-REP.
