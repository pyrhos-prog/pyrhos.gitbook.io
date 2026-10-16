# Perturbaciones y ancho de banda

### Perturbaciones de la señal

| Fenómeno               | Qué es                                                                 | Dónde aparece                  | Cómo se mitiga                                                   |
| ---------------------- | ---------------------------------------------------------------------- | ------------------------------ | ---------------------------------------------------------------- |
| Atenuación             | Pérdida de intensidad con la distancia                                 | Todos los medios               | Respetar longitudes, repetidores, regeneradores o amplificadores |
| Distorsión por retardo | Los componentes de la señal llegan en momentos distintos               | Medios guiados                 | Buen diseño del enlace y del medio                               |
| Ruido                  | Señales no deseadas sumadas (térmico, interferencias de otros equipos) | Todos                          | Apantallamiento, separar de fuentes de ruido                     |
| Diafonía               | Un conductor interfiere en otro cercano                                | Par trenzado                   | Trenzado, pantalla, buena terminación                            |
| Absorción atmosférica  | Oxígeno, vapor de agua, lluvia o niebla reducen la señal               | Inalámbrico, frecuencias altas | Margen de potencia, frecuencias adecuadas                        |
| Multitrayecto          | La señal llega por varios caminos tras rebotar                         | Inalámbrico                    | Wi-Fi usa OFDM y MIMO para aprovecharlo o reducirlo              |

### Diagnóstico según síntoma

| Síntoma                                           | Sospechar                                                 |
| ------------------------------------------------- | --------------------------------------------------------- |
| Enlace estable pero lento                         | Terminación defectuosa, diafonía o categoría insuficiente |
| Cae con la distancia                              | Atenuación: se supera el límite del medio                 |
| Falla cerca de cables de potencia o fluorescentes | Ruido e interferencia EM                                  |
| Wi-Fi inestable en un punto concreto              | Multitrayecto o interferencia de otros APs                |

### Ancho de banda, throughput y goodput

| Término                   | Significado                                                                   | Unidad         |
| ------------------------- | ----------------------------------------------------------------------------- | -------------- |
| Ancho de banda (físico)   | Intervalo de frecuencias que ocupa la señal (f máx - f mín)                   | Hz             |
| Ancho de banda (en redes) | Capacidad máxima teórica de un enlace                                         | Mbit/s, Gbit/s |
| Throughput                | Bits realmente transferidos por segundo                                       | Mbit/s         |
| Goodput                   | Solo datos útiles para el usuario (sin cabeceras, retransmisiones ni control) | Mbit/s         |

```
Goodput <= Throughput <= Capacidad nominal
```

Una conexión de 1 Gbit/s no descargará a 1 Gbit/s: pesan protocolos, congestión, errores y límites de los equipos.

#### Ejemplo

Fibra de 300 Mbit/s con 3 PCs descargando a la vez: en igualdad de condiciones, unos 100 Mbit/s cada uno como máximo. En la práctica depende del servidor remoto, de procesos que consuman red y de otros factores.

### Medir

```bash
# Linux: test de velocidad entre dos equipos de tu red
iperf3 -s                    # en el servidor
iperf3 -c 192.168.1.10 -t 10 # en el cliente

# Velocidad negociada de la interfaz
ethtool eth0 | grep -E "Speed|Duplex"
```

```powershell
# Windows: velocidad negociada del adaptador
Get-NetAdapter | Select Name, LinkSpeed, Status
```

{% hint style="warning" %}
Al hacer un test de velocidad contra Internet, conviene que no haya otros equipos consumiendo ancho de banda, o se restará del resultado.
{% endhint %}
