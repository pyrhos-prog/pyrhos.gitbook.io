# Wireshark

Wireshark es una herramienta para analizar el tráfico de una red, en hacking web nos va a servir para analizar y comprender el trafico.

### Funciones en hacking web

* Capturar trafico de red.
* Analizar tramas Wi-Fi.
* Identificar handshakes, tramas de control y datos.
* Depurar problemas de la red inalámbrica.

### Requisitos para analizar el tráfico

* Adaptador Wi-Fi con modo monitor
* Realizar la captura del tráfico con otras herramientas en modo monitor.
* Capturas del tráfico guardadas en formato <kbd>.pcap</kbd> o <kbd>.pcapng</kbd> .



### Analisis de la captura

#### 1. Abrir la captura

```
wireshark captura.pcap
```

#### 2. Filtrar la captura para encontrar la información que buscamos

| Filtro                                     | uso                                                      |
| ------------------------------------------ | -------------------------------------------------------- |
| <kbd>wlan</kbd>                            | Muestra solo el trafico Wi-Fi                            |
| <kbd>wlan.fc.type == 0</kbd>               | Filtra por tráfico de gestión                            |
| <kbd>wlan.fc.type == 1</kbd>               | Filtra por tráfico de control                            |
| <kbd>wlan.fc.type == 2</kbd>               | Filtra por tráfico de datos                              |
| <kbd>eapol</kbd>                           | Muestra los paquetes del 4-way handshake                 |
| <kbd>wlan.bssid == AA:BB:CC:DD:EE:FF</kbd> | Filtra por el tráfico de un AP a través de la mac        |
| <kbd>wlan.sa == AA:BB:CC:DD:EE:FF</kbd>    | Filtra por el tráfico de una estación a través de la mac |
