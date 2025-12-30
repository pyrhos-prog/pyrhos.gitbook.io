# Tramas Wi-Fi

Las comunicaciones Wi-Fi se basan en tramas, las tramas son la unidad básica de la información que se transmite entre dispositivos por el aire.

**Existen 3 tipos diferentes de tramas:**

* Gestión&#x20;
* Control
* Data



## Tramas

Las tramas:&#x20;

* Transportan información entre dispositivos
* Incluye direcciones MAC, tipo de mensajes y datos
* Es equivalente a un paquete pero a nivel Wi-Fi (capa 2)

Las tramas permite:

* Detectar redes
* Conectarse a las redes
* Enviar y recibir datos



### Tramas de gestión

Se encargan de la gestión de la red

|                           |                                                                                                               |
| ------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Funciones principales** | <ul><li>Descubrir redes</li><li>Autenticarse</li><li>Asociarse a un AP</li><li>Mantener la conexión</li></ul> |
| **Tramas de gestión**     | <ul><li>Beacon</li><li>Probe Request</li><li>Authentication</li><li>Deauthentication</li></ul>                |
| **Características**       | <ul><li>Se envian sin estar conectado</li><li>Muchas no van cifradas</li></ul>                                |

#### Beacon

El AP envia de beacons de forma periódica.

**Contienen**

* ESSID
* BSSID
* Canal
* Tipo de cifrado

**Sirven para**

* Anunciar la existencia de la red
* Permitir que los clientes lo deteten

#### Probe Request y Probe Response

* **Probe request:** La Station Busca redes
* **Probe response:** el AP responde

**Se usan cuando**

* El cliente escanea activamente
* La red no emite beacons visibles&#x20;



### Tramas de control

Se usan para coodinar la transmisión y evitar colisiones.

|                           |                                                                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Funciones principales** | <ul><li>Confirmar recepción</li><li>Controlar el acceso al medio</li><li>Optimiza el tráfico</li></ul>                               |
| **Tramas de control**     | <ul><li>ACK</li><li>RTS</li><li>CTS</li></ul>                                                                                        |
| **Características**       | <ul><li>No transporta datos de usuario</li><li>Mejora la eficiencia de la red</li><li>Son invisibles para el usuario final</li></ul> |

#### ACK

Confirma que una trama fue recibida correctamente.

* El receptor envía un ACK al emisor.
* Indica que la trama llegó sin errores.
* Si no se recibe ACK, la trama se retransmite.

#### RTS

Es una solicitud para enviar datos.

* El emisor pregunta si el canal está libre.
* Se usa antes de transmitir tramas grandes.
* Reduce colisiones en redes con muchos dispositivos.

#### CTS

Es la respuesta al RTS

* El receptor autoriza el envío de datos.
* Informa a otros dispositivos de que el canal estará ocupado.
* Reserva el medio durante un tiempo determinado.



### Tramas de datos&#x20;

Transportan los datos reales del usuario.

|                           |                                                                                                                                                                 |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Funciones principales** | <ul><li>Transportar la información real del usuario </li><li>Encapsular datos de capas superiores</li><li>Permitir la comunicaicion entre STA y AP</li></ul>    |
| **Tramas de datos**       | <ul><li>QoS Data</li><li>Null Data</li><li>QoS Null</li></ul>                                                                                                   |
| **Características**       | <ul><li>Pueden ir cirfradas</li><li>Soportan control de calida de servicios</li><li>Requieren confirmaciones mediante ACK</li><li>Pueden fragmentarse</li></ul> |



#### QoS Data

Trama de datos con control de calidad de servicio

* Prioriza cierto tipo de tráfico
* Se usa para voz y vídeo (Llamadas VoIP, Streaming, Videoconferencias...)&#x20;
* Reduce latencia y cortes

#### Null Data

Trama de datos sin carga útil

* No transportan información el usuario
* Se usa para control de estado
* indica cambios de energía o conexión

#### QoS Null

Variate de Null Data con soporte QoS

* Mantiene sincronización con el AP
* Se usa en redes modernas
* Optimiza el consumo energético











