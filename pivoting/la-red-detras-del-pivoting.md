# La red detrás del pivoting

## Creación de redes detrás del Pivoting — Apuntes

### Contexto general

El pivoting depende directamente de fundamentos de redes. Para aplicarlo correctamente es necesario comprender direccionamiento IP, interfaces de red, enrutamiento y puertos/protocolos.

***

### Direccionamiento IP y NIC

Cada equipo en una red necesita una dirección IP para comunicarse. Esta puede asignarse de forma dinámica mediante DHCP o de forma estática en dispositivos como servidores, routers, impresoras o infraestructura crítica.

La IP siempre está asociada a una interfaz de red (NIC). Una NIC puede ser física o virtual, y un mismo sistema puede tener varias NICs simultáneamente, cada una con una o varias direcciones IP.

Esto es relevante en pivoting porque la presencia de múltiples interfaces suele implicar acceso a distintas redes internas.

Herramientas básicas para inspección:

* Linux/macOS: `ifconfig`
* Windows: `ipconfig`

***

### Interfaces de red y significado operativo

En sistemas Linux es habitual observar múltiples interfaces:

* Interfaces Ethernet como `eth0` o `eth1`, asociadas a redes distintas
* `lo` como interfaz de loopback local (127.0.0.1)
* `tun0` como interfaz virtual de VPN

La existencia de varias interfaces indica conectividad simultánea a diferentes segmentos de red.

Las direcciones IP pueden ser de dos tipos principales:

* IP pública: accesible desde Internet, normalmente en sistemas expuestos o DMZ
* IP privada: utilizada dentro de redes internas, no enrutable directamente en Internet

En entornos reales, la traducción entre ambos tipos se realiza mediante NAT, que convierte tráfico entre redes privadas y públicas.

***

### Interfaces en Windows

En Windows, `ipconfig` muestra los adaptadores disponibles. Es habitual encontrar múltiples interfaces, incluyendo adaptadores físicos, virtuales y VPN.

Un sistema puede operar en configuración dual stack, con IPv4 e IPv6 simultáneamente.

El gateway predeterminado es la ruta de salida hacia otras redes. Normalmente corresponde a un router interno.

***

### Máscara de subred

La máscara de subred define qué parte de una dirección IP corresponde a la red y qué parte al host.

Su función principal es determinar si un destino está dentro de la misma red o en una red diferente.

* Si está en la misma red → comunicación directa
* Si está en otra red → tráfico enviado al gateway

***

### Enrutamiento

El enrutamiento determina cómo se reenvía el tráfico entre redes.

Los sistemas mantienen una tabla de rutas que indica:

* Redes conocidas directamente
* Gateways para redes externas
* Ruta por defecto (default route)

Si no existe una ruta específica, el tráfico se envía a la ruta por defecto.

En pivoting, la tabla de enrutamiento es crítica porque define qué redes son alcanzables desde un host comprometido y qué rutas adicionales pueden necesitarse.

***

### Puertos, protocolos y servicios

Los protocolos definen cómo se comunican los sistemas en red. Cada servicio expuesto normalmente está asociado a un puerto lógico.

* La dirección IP identifica el host
* El puerto identifica el servicio dentro del host

Ejemplo: HTTP suele operar en el puerto 80.

Los puertos abiertos representan puntos de interacción con servicios que pueden estar permitidos por firewall, lo que puede facilitar el acceso inicial o movimiento dentro de una red.

También existe el puerto de origen, usado para rastrear conexiones desde el lado cliente.

***

### Relación con pivoting

El pivoting depende directamente de estos conceptos:

* Múltiples NICs → acceso a múltiples redes
* Máscara de subred → define límites de red
* Gateway → salida hacia otras redes
* Tabla de rutas → define caminos disponibles
* Puertos → acceso a servicios internos

La identificación de estas propiedades en hosts comprometidos permite determinar posibles rutas de acceso hacia redes no directamente alcanzables.
