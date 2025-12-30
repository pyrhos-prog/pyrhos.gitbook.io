# Que es el Wi-Fi

Wifi es el nombre que recibe el grupo de protocolos de red inalámbrica que permiten a los dispositivos conectarse de forma inalámbrica a un router para poder acceder a internet. Los protocolos Wi-Fi se basan en la familia de normas IEEE 802.11

### Estandar IEEE 802.11

Son el conjunto de estándares que definen las reglas en la redes Wi-Fi, especifica aspectos como la transmisión de datos por radiofrecuencia, el acceso al medio, la autentificación y el formato de las tramas.&#x20;

Cada revisión del estándar introduce mejoras en velocidad, eficiencia o gestión del espectro:

* 802.11b/g - 2.4GHz
* 802.11a/n/ac - 5Ghz
* 802.11ax - 2.4 GHz y 5 Ghz&#x20;

### Diferencia entre red cableada y red inalámbrica

| Red cableada           | Red inalámbrica           |
| ---------------------- | ------------------------- |
| Medio físico           | Medio compartido          |
| Difícil de interceptar | Fácil de capturar         |
| Menor interferencia    | Sensible a interferencias |
| Mayor estabilidad      | Menor estabilidad         |



### Que sucede al conectarse a una red Wi-Fi

El proceso de conexión a una red Wi-Fi sigue una secuencia definida:

{% stepper %}
{% step %}
### Escaneo

El cliente busca redes disponibles.
{% endstep %}

{% step %}
### Autenticación

Intercambio incial de tramas con el AP.
{% endstep %}

{% step %}
### Asociación

El AP acepta al cliente en la red.
{% endstep %}

{% step %}
### Handshake de seguridad

Negociación de claves (WPA, WPA2, WPA3).
{% endstep %}

{% step %}
### Asignación de IP

Normalmente mediante DHCP.
{% endstep %}

{% step %}
### Transmisión de datos

Comienza el tráfico cifrado.
{% endstep %}
{% endstepper %}

