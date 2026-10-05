---
icon: wifi
---

# Modo monitor e inyección de paquetes

> El modo monitor o modo promiscuo es un modo de funcionamiento de una tarjeta de red inalámbrica que permite escuchar todos los paquetes que hay en el aire. No se limita a capturar solo los datos dirigidos a nuestro equipo, sino que registra todo el intercambio de información de las redes Wi-Fi al alcance.

### Funciones Principales

* **Capturar paquetes:** Analizar el tráfico completo del espectro.
* **Identificar dispositivos:** Ver las direcciones MAC de los dispositivos cliente y puntos de acceso alrededor.
* **Capturar tramas Wi-Fi:** Esencial para auditorías, captura de _handshakes_ e inyección de paquetes.

{% hint style="info" %}
Compatibilidad Dependiendo del chipset de la tarjeta Wi-Fi, se podrá o no usar el modo monitor y la inyección de paquetes. No todas las tarjetas integradas en portátiles lo soportan.
{% endhint %}

### Cómo Activar y Desactivar el Modo Monitor

Antes de empezar, siempre es necesario identificar el nombre lógico que el sistema le ha asignado a nuestra tarjeta de red (por ejemplo, `wlan0`, `wlp2s0`, etc.).

```
# Ver las interfaces inalámbricas disponibles
iwconfig
# o también
iw dev

```

#### Método 1: Usando `airmon-ng`&#x20;

Este es el método recomendado para auditorías, ya que la herramienta gestiona automáticamente los procesos conflictivos y suele crear una interfaz virtual específica para el modo monitor.

**Activar el modo monitor:**

```
# 1. Matar procesos que puedan interferir (NetworkManager, wpa_supplicant...)
sudo airmon-ng check kill

# 2. Activar el modo monitor en la interfaz
sudo airmon-ng start wlan0

```

**Desactivar el modo monitor:**

```
# 1. Detener la interfaz en modo monitor
sudo airmon-ng stop wlan0mon

# 2. Reiniciar el gestor de red para recuperar la conexión a Internet
sudo systemctl start NetworkManager

```

#### Método 2: Usando `iw` y manual (Nativo en Linux)

Este método es más puro y no depende de la suite de Aircrack. Es ideal para configuraciones de red estándar o scripting.

**Activar el modo monitor:**

```
# 1. Bajar la interfaz para poder modificarla
sudo ip link set wlan0 down

# 2. Cambiar el modo de operación a "monitor"
sudo iw dev wlan0 set type monitor

# 3. Volver a levantar la interfaz
sudo ip link set wlan0 up

```

**Desactivar el modo monitor y volver a modo "managed" (cliente):**

```
# 1. Bajar la interfaz
sudo ip link set wlan0 down

# 2. Cambiar el modo de operación a "managed" (modo estación normal)
sudo iw dev wlan0 set type managed

# 3. Levantar la interfaz
sudo ip link set wlan0 up

# 4. (Opcional) Reiniciar el servicio de red si no se conecta automáticamente
sudo systemctl restart NetworkManager
```
