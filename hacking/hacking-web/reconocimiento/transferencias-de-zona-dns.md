# Transferencias de zona DNS

Si bien la fuerza bruta puede ser un método fructífero, existe un método menos invasivo y potencialmente más eficiente para descubrir subdominios: las transferencias de zona DNS. Este mecanismo, diseñado para replicar registros DNS entre servidores de nombres, puede convertirse inadvertidamente en una mina de oro de información para intrusos si se configura incorrectamente.

### ¿Qué es una transferencia de zona?

Una transferencia de zona DNS consiste básicamente en una copia completa de todos los registros DNS dentro de una zona (un dominio y sus subdominios) de un servidor de nombres a otro. Este proceso es esencial para mantener la coherencia y la redundancia entre los servidores DNS. Sin embargo, si no se protege adecuadamente, terceros no autorizados pueden descargar el archivo de zona completo, revelando una lista completa de subdominios, sus direcciones IP asociadas y otros datos DNS confidenciales.

![Diagrama que muestra la transferencia de datos entre los servidores secundario y principal. Incluye los pasos: Solicitud XML, Registro XML, bucle de reintentos, Informe XML y Confirmación de Aceptación (AOK).](<../.gitbook/assets/pakoeNqNkc9qwzAMxl9F JSx7gV8KISWXcY2aHYYwxdjK39obGWKvBFK333ukg5aGNQnW9b3Q_q g3LkUWk14mfC6HDb2YZtMBHyGdFR9JanCvkL WG9vh 4C38FDeX74w52J 0oUHxQRHhjG8ca W5mXAgy4YqpoXotM8EReygqsSxANZRJWuJOpoXSEw0gC3ku3QTfvlQLfBZh9DeOdbELbCgMPQr 58u1LZsnKEq3j_Tdo28wYJS8iVqpgBx>)

{% stepper %}
{% step %}
### Zone Transfer Request (AXFR)

El servidor DNS secundario inicia el proceso enviando una solicitud de transferencia de zona al servidor principal. Esta solicitud suele ser del tipo AXFR (Transferencia de Zona Completa).
{% endstep %}

{% step %}
### SOA Record Transfer

Tras recibir la solicitud (y posiblemente autenticar el servidor secundario), el servidor principal responde enviando su registro de inicio de autoridad (SOA). Este registro contiene información vital sobre la zona, incluido su número de serie, lo que ayuda al servidor secundario a determinar si los datos de su zona están actualizados.
{% endstep %}

{% step %}
### DNS Records Transmission

El servidor principal transfiere todos los registros DNS de la zona al servidor secundario, uno por uno. Esto incluye registros como A, AAAA, MX, CNAME, NS y otros que definen los subdominios, servidores de correo, servidores de nombres y otras configuraciones del dominio.
{% endstep %}

{% step %}
### Zone Transfer Complete

Una vez transmitidos todos los registros, el servidor principal señala el final de la transferencia de zona. Esta notificación informa al servidor secundario que ha recibido una copia completa de los datos de zona.
{% endstep %}

{% step %}
### Acknowledgement (ACK)

El servidor secundario envía un mensaje de confirmación al servidor principal, confirmando la recepción y el procesamiento correctos de los datos de zona. Esto completa el proceso de transferencia de zona.
{% endstep %}
{% endstepper %}

### La vulnerabilidad de transferencia de zona

Si bien las transferencias de zona son esenciales para la gestión legítima del DNS, un servidor DNS mal configurado puede convertir este proceso en una vulnerabilidad de seguridad importante. El problema principal reside en los controles de acceso que rigen quién puede iniciar una transferencia de zona.

En los inicios de Internet, era común que cualquier cliente solicitara una transferencia de zona desde un servidor DNS. Este enfoque abierto simplificaba la administración, pero abría una brecha de seguridad considerable. Esto implicaba que cualquiera, incluso actores maliciosos, podía solicitar a un servidor DNS una copia completa de su archivo de zona, que contiene gran cantidad de información confidencial.

La información obtenida de una transferencia de zona no autorizada puede ser invaluable para un atacante. Revela un mapa completo de la infraestructura DNS del objetivo, que incluye:

* Subdomains: Una lista completa de subdominios, muchos de los cuales podrían no estar enlazados desde el sitio web principal ni ser fácilmente detectables por otros medios. Estos subdominios ocultos podrían albergar servidores de desarrollo, entornos de prueba, paneles administrativos u otros recursos confidenciales.
* IP Addresses: Las direcciones IP asociadas a cada subdominio, que proporcionan objetivos potenciales para futuros reconocimientos o ataques.
* Name Server Records: Detalles sobre los servidores de nombres autorizados para el dominio, revelando el proveedor de alojamiento y posibles configuraciones erróneas.

{% hint style="warning" %}
La exposición de un archivo de zona completo puede permitir a un atacante mapear toda la infraestructura DNS del objetivo. Trátese como una vulnerabilidad seria y priorice la corrección si se detecta.
{% endhint %}

#### **Remediación**

Afortunadamente, la concienciación sobre esta vulnerabilidad ha aumentado y la mayoría de los administradores de servidores DNS han mitigado el riesgo. Los servidores DNS modernos suelen estar configurados para permitir transferencias de zona únicamente a servidores secundarios de confianza, lo que garantiza la confidencialidad de los datos sensibles de la zona.

Sin embargo, aún pueden producirse configuraciones incorrectas debido a errores humanos o prácticas obsoletas. Por ello, intentar una transferencia de zona (con la debida autorización) sigue siendo una valiosa técnica de reconocimiento. Incluso si no tiene éxito, el intento puede revelar información sobre la configuración y la seguridad del servidor DNS.

{% hint style="success" %}
Práctica recomendada: restringir las transferencias de zona a direcciones IP específicas de servidores secundarios autorizados y auditar regularmente las configuraciones de los servidores DNS.
{% endhint %}

**Transferencias de zonas de explotación**

Puede utilizar el comando dig para solicitar una transferencia de zona:

{% code title="Ejemplo" %}
```shell
pyrhos@htb[/htb]$ dig axfr @nsztm1.digi.ninja zonetransfer.me
```
{% endcode %}

Este comando indica que dig debe solicitar una transferencia de zona completa (axfr) al servidor DNS responsable de zonetransfer.me. Si el servidor está mal configurado y permite la transferencia, recibirá una lista completa de registros DNS del dominio, incluidos todos los subdominios.

{% code title="Salida de ejemplo" %}
```shell
pyrhos@htb[/htb]$ dig axfr @nsztm1.digi.ninja zonetransfer.me

; <<>> DiG 9.18.12-1~bpo11+1-Debian <<>> axfr @nsztm1.digi.ninja zonetransfer.me
; (1 server found)
;; global options: +cmd
zonetransfer.me.	7200	IN	SOA	nsztm1.digi.ninja. robin.digi.ninja. 2019100801 172800 900 1209600 3600
zonetransfer.me.	300	IN	HINFO	"Casio fx-700G" "Windows XP"
zonetransfer.me.	301	IN	TXT	"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me.	7200	IN	MX	0 ASPMX.L.GOOGLE.COM.
...
zonetransfer.me.	7200	IN	A	5.196.105.14
zonetransfer.me.	7200	IN	NS	nsztm1.digi.ninja.
zonetransfer.me.	7200	IN	NS	nsztm2.digi.ninja.
_acme-challenge.zonetransfer.me. 301 IN	TXT	"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me. 14000 IN	SRV	0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me. 7200	IN PTR www.zonetransfer.me.
asfdbauthdns.zonetransfer.me. 7900 IN	AFSDB	1 asfdbbox.zonetransfer.me.
asfdbbox.zonetransfer.me. 7200	IN	A	127.0.0.1
asfdbvolume.zonetransfer.me. 7800 IN	AFSDB	1 asfdbbox.zonetransfer.me.
canberra-office.zonetransfer.me. 7200 IN A	202.14.81.230
...
;; Query time: 10 msec
;; SERVER: 81.4.108.41#53(nsztm1.digi.ninja) (TCP)
;; WHEN: Mon May 27 18:31:35 BST 2024
;; XFR size: 50 records (messages 1, bytes 2085)
```
{% endcode %}

> zonetransfer.me es un servicio configurado específicamente para demostrar los riesgos de las transferencias de zona de modo que el dig devuelva el registro de zona completo.
