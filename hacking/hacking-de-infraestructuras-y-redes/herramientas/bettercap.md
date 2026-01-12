# Bettercap

> Es un framework modular para ataques de red y auditoría de tráfico que permite interceptar, modificar y analizar tráfico en tiempo real.

### Usos principales

* Interceptar comunicaciones en redes locales
* Analizar protocolos en texto plano
* Ejecutar ataques MITM
* Auditar redes Wi-Fi y Ethernet
* ARP spoofing en redes LAN
* Sniffing de credenciales
* Ataques a protocolos no cifrados
* Auditoría de redes Wi-Fi
* Análisis de tráfico en tiempo real

### Arquitectura

Bettercap funciona mediante **módulos** que se activan de forma independiente. Cada módulo realiza una tarea concreta (sniffing, spoofing, reconexiones, etc.).

### Interfaz

Bettercap dispone de:

* Interfaz interactiva (CLI)
* Interfaz web (`caplet`)
* Sistema de scripts automatizados

### Módulos principales

| `net.probe`   | Descubre hosts en la red |
| ------------- | ------------------------ |
| `arp.spoof`   | Ataque MITM por ARP      |
| `net.sniff`   | Captura tráfico          |
| `http.proxy`  | Proxy HTTP               |
| `dns.spoof`   | Envenenamiento DNS       |
| `wifi.recon`  | Reconocimiento Wi-Fi     |
| `wifi.assoc`  | Asociación a AP          |
| `wifi.deauth` | Desautenticación         |

### Ejemplo básico de uso

Inicio interactivo:

```bash
bettercap
```

Escaneo de red:

```bash
net.probe on
```

### Ejemplo de ataque MITM (ARP spoofing)

```bash
set arp.spoof.targets 192.168.1.0/24
arp.spoof on
net.sniff on
```

### Reconocimiento Wi-Fi

```bash
wifi.recon on
```

Muestra información de las redes Wi-Fi:

* ESSID
* BSSID
* Canal
* Clientes asociados
* Tipo de cifrado

### Caplets

Los **caplets** son scripts que automatizan acciones.

Ejemplo:

```bash
bettercap -caplet http-ui
```

Permite gestionar bettercap desde navegador web.

### Relación con otras herramientas

| Herramienta | Enfoque                      |
| ----------- | ---------------------------- |
| bettercap   | MITM en tiempo real          |
| Wireshark   | Análisis pasivo              |
| hcxtools    | Captura para ataques offline |
| aircrack-ng | Auditoría clásica Wi-Fi      |
