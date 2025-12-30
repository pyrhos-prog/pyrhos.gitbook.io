# Hardware para hacking Wi-Fi

Además de las tarjetas Wi-Fi, existen mas herramientas físicas para auditar redes Wi-Fi, como por ejemplo estas:



### WiFi Pineapple (Hak5)

{% embed url="https://www.youtube.com/watch?v=_s5NDXYY-Z8" %}

#### Descripción

Dispositivo dedicado a auditoría Wi-Fi que funciona de forma autónoma. Está orientado a ataques controlados y automatizados en entornos de pentesting.

#### Características

* Plataforma independiente (no requiere portátil)
* Soporte para ataques Evil Twin
* Captura de tráfico y credenciales
* Interfaz web de gestión

### Flipper Zero

{% embed url="https://youtu.be/jQOCzk2HDOM?si=QdIidqTkhN7Op8Rb" %}

#### Descripción

Dispositivo portátil multifunción enfocado a pruebas de seguridad inalámbrica. En Wi-Fi depende de un módulo externo basado en ESP32.

#### Características

* Módulo Wi-Fi adicional
* Escaneo básico de redes
* Sniffing limitado
* Alta portabilidad



### M5Stick

{% embed url="https://youtu.be/Lphkt7DQ8s0?si=YIGJHp1BHEt2sceG" %}

#### Descripción

Placa compacta basada en ESP32 con pantalla y batería integrada, utilizada en proyectos experimentales de análisis Wi-Fi.

#### Características

* Wi-Fi integrado
* Pantalla para visualización en tiempo real
* Soporte para firmware personalizado
* Uso habitual en wardriving y escaneo pasivo

### ESP32 Marauder

{% embed url="https://youtu.be/lcokJQMivwY?si=FIr671ddN9w8WD39" %}

#### Descripción

Firmware para ESP32 orientado a ataques y análisis Wi-Fi y Bluetooth.

#### Características

* Escaneo de redes Wi-Fi
* Ataques de desautenticación
* Captura parcial de tráfico
* Interfaz por pantalla o consola
* No permite cracking WPA real

### T-Embed

{% embed url="https://youtu.be/u-uCiy7gFHE?si=qdAk1CoLZtRcyzlZ" %}

#### Descripción

Dispositivo basado en ESP32 con pantalla integrada, pensado para desarrollo rápido y visualización directa de información inalámbrica.

#### Características

* Wi-Fi integrado
* Pantalla a color
* Compatible con firmwares como Marauder
* Portátil y autónomo



### Comparación

| Dispositivo              | Tipo                | Capacidades Wi-Fi               | Ataques activos | Sniffing | Portabilidad | Nivel         |
| ------------------------ | ------------------- | ------------------------------- | --------------- | -------- | ------------ | ------------- |
| **WiFi Pineapple**       | Plataforma dedicada | Evil Twin, MITM, automatización | Sí              | Sí       | Media        | Profesional   |
| **Flipper Zero + Wi-Fi** | Multifunción        | Escaneo básico                  | No              | Limitado | Muy alta     | Introductorio |
| **M5Stick**              | ESP32               | Escaneo, wardriving             | Limitado        | Parcial  | Muy alta     | Experimental  |
| **ESP32 Marauder**       | Firmware ESP32      | Deauth, escaneo                 | Limitado        | Parcial  | Alta         | Educativo     |
| **T-Embed**              | ESP32 con pantalla  | Escaneo, visualización          | Limitado        | Parcial  | Alta         | Experimental  |
