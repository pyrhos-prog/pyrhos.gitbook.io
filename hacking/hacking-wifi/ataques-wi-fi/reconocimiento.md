# Reconocimiento

El reconocimiento en hacking Wi-Fi es la fase en la que se **recopila información de las redes inalámbricas** disponibles sin interactuar directamente con ellas.<br>

### Tipo de reconocimiento

#### Reconocimiento pasivo

No se envian paquetes, solo se capturan tramas emitidas por los dipositivos y puntos de acceso cercanos.

**Información obtenida**

* ESSID
* BSSID
* Canal
* Tipo de cifrado
* Clientes

#### Reconocimiento activo

Se envian tramas a los clientes y los puntos de acceso.

**Información obtenida adicionalmente**

* Handshakes
* PMKID
* Repuestas del AP
* Redes ocultas

### Herramientas habituales

#### airodump-ng

```bash
airodump-ng [interfaz]
```

* Permite ver APs
* Detecta clientes
* Captura handshakes

#### hcxdumptool

```bash
hcxdumptool -i [interfaz] -o captura.pcapng
```

* Captura handshakes
* Tramas de gestión&#x20;
* PMKID

#### bettercap

```bash
wifi.recon on
```

Muestra:

* Redes
* Clientes
* Cifrado
* Canales

### Fases del reconocimiento

{% hint style="info" %}
Para poder ejecutar el reconocimiento hay que tener la antena en modo monitor.
{% endhint %}

#### 1. Descubrimiento de redes

```bash
airodump-ng [interfaz]
```

#### 2. Clasificación del objetivo

Filtrado por canal

```bash
airodump-ng -c [canal] [interfaz]
```

#### 3. Enumeración de clientes

Identificación de dispositivos asociados al AP.

```bash
airodump-ng -c [canal] -bssid AA:BB:CC:DD:EE:FF [interfaz]
```

#### 4. Analisis de cifrado y autenticación

Comprobar el método de seguridad

```bash
airodump-ng --bssid AA:BB:CC:DD:EE:FF -c [canal] [interfaz]
```

#### 6. Captura del tráfico relevante

Guardar tráfico para el  analisis posterior

```bash
airodump-ng --bssid AA:BB:CC:DD:EE:FF -C [canal] -w captura [interfaz]
```

#### 7. Captura del PMKID

Obtener material sin necesidad de clientes activos.

```
hcxdumptool -i [interfaz] -o captura.pcapng --enable_status=1
```

#### 8. Procesamiento de capturas

Extraer hashes útiles

```
hcxpcapngtool -o hash.hc22000 captura.pcapng
```

Verificación:

```
hcxpcapngtool -I captura.pcapng
```
