# Ethernet y dominios de colisión

## Ethernet

> Ethernet es una **tecnología de LAN** estandarizada por **IEEE 802.3**. Define cómo se conectan los equipos por cable y cómo se envían los datos entre ellos en una red local.

### Dónde encaja en el modelo OSI

Ethernet trabaja en las **dos capas inferiores**:

| Capa          | Qué define Ethernet                                                                           |
| ------------- | --------------------------------------------------------------------------------------------- |
| **1. Física** | Medio (cobre, fibra), conectores (RJ45), señales y velocidades (10/100/1000 Mbps, 10 Gbps...) |
| **2. Enlace** | Direcciones **MAC** y formato de las **tramas**                                               |

### Subcapas de enlace en IEEE 802

IEEE divide la capa de enlace en dos subcapas:

| Subcapa         | Función                                           | ¿Común a todas las tecnologías? |
| --------------- | ------------------------------------------------- | ------------------------------- |
| **LLC (802.2)** | Interfaz hacia la capa de red                     | Sí                              |
| **MAC**         | Direccionamiento y **control de acceso al medio** | No, depende de la tecnología    |

#### Tecnologías LAN que usan este esquema

| Estándar   | Tecnología | Medio               |
| ---------- | ---------- | ------------------- |
| **802.3**  | Ethernet   | Cable (cobre/fibra) |
| **802.11** | Wi-Fi      | Radio               |

Comparten LLC, pero cambian la subcapa MAC y la capa física.

### La trama Ethernet (Ethernet II)

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

La trama mínima es de 64 bytes. Si hay menos datos, se rellena (_padding_).

### Dominio de colisión y control de acceso al medio

uno es el problema y el otro la solución.

#### El problema: dominio de colisión

Parte de la red donde **dos equipos pueden transmitir a la vez** y provocar una colisión.

#### La solución: CSMA (Carrier Sense Multiple Access)

La idea base es **escuchar antes de transmitir**.

| Variante                          | Dónde se usa                  | Qué hace                                                                                     |
| --------------------------------- | ----------------------------- | -------------------------------------------------------------------------------------------- |
| **CSMA/CD** (Collision Detection) | Ethernet cableado half-duplex | Detecta la colisión mientras transmite                                                       |
| **CSMA/CA** (Collision Avoidance) | Wi-Fi                         | Intenta evitarla, porque detectar una colisión mientras se transmite es muy difícil en radio |

**Funcionamiento de CSMA/CD:**

1. Escucha el medio.
2. Si está libre, transmite.
3. Si detecta colisión, envía una señal de _jam_.
4. Espera un tiempo aleatorio (_backoff_ exponencial) y reintenta.

#### Dominios por dispositivo

Con **switches y full-duplex**, las colisiones prácticamente desaparecen y CSMA/CD ya casi no se usa.

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

El switch separa dominios de colisión, pero **no** de broadcast. Eso lo hace el router (o las VLANs).

### Ver el enlace y la MAC

```bash
# Linux
ip -br link          # interfaces y MAC
ethtool eth0         # velocidad, duplex, enlace
ethtool -S eth0      # contadores y errores
ip neigh             # tabla ARP
```

```powershell
# Windows
getmac /v            # MAC de cada adaptador
arp -a               # tabla ARP
```

### Seguridad

| Dispositivo | Comportamiento                                    | Riesgo                                                                                 |
| ----------- | ------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Hub**     | Reenvía todo a todos los puertos                  | Cualquier equipo conectado puede capturar el tráfico de los demás (_sniffing_ trivial) |
| **Switch**  | Aprende MACs y reenvía solo por el puerto destino | Mejor, pero **no es una defensa completa**                                             |

**Ataques habituales contra un switch:**

* **MAC flooding**: satura la tabla CAM y el switch pasa a reenviar a todos los puertos.
* **ARP spoofing**: envenena la caché ARP para hacer _man-in-the-middle_.
* **VLAN hopping** y manipulación de **STP**.

**Defensas:** port security, DHCP snooping, Dynamic ARP Inspection, 802.1X y segmentación con VLANs.
