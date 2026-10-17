# Wi-Fi: estándares, bandas y canales

Wi-Fi es el nombre comercial de las redes basadas en la familia IEEE 802.11.

### Estándares

| Nombre comercial | IEEE     | Bandas         | Novedades                                              |
| ---------------- | -------- | -------------- | ------------------------------------------------------ |
| Wi-Fi 4          | 802.11n  | 2,4 y 5 GHz    | MIMO; canales de 20 o 40 MHz                           |
| Wi-Fi 5          | 802.11ac | 5 GHz          | Canales de 20, 40, 80 y hasta 160 MHz                  |
| Wi-Fi 6          | 802.11ax | 2,4 y 5 GHz    | OFDMA y MU-MIMO; más eficiente con muchos clientes     |
| Wi-Fi 6E         | 802.11ax | Añade 6 GHz    | Más espectro y menos congestión, donde esté autorizada |
| Wi-Fi 7          | 802.11be | 2,4, 5 y 6 GHz | Canales de hasta 320 MHz, Multi-Link Operation         |

### Qué banda usar

| Banda   | Alcance                           | Velocidad y capacidad | Congestión           | Usar para                             |
| ------- | --------------------------------- | --------------------- | -------------------- | ------------------------------------- |
| 2,4 GHz | Mayor, atraviesa mejor obstáculos | Menor                 | Alta (pocos canales) | Cobertura, IoT, dispositivos antiguos |
| 5 GHz   | Menor                             | Mayor                 | Media (más canales)  | Portátiles, móviles, streaming        |
| 6 GHz   | Aún menor                         | Mayor, más espectro   | Baja                 | Dispositivos compatibles cerca del AP |

{% hint style="info" %}
Cada banda tiene ventajas y desventajas, no hay una mejor que otra, 2,4 GHz favorece la cobertura y 5/6 GHz la capacidad.
{% endhint %}

### Canales

* En 2,4 GHz los canales están muy próximos y se solapan. En Europa se usan del 1 al 13.
* Con canales de 20 MHz, los tres sin solape son 1, 6 y 11.
* En 5 GHz hay más canales; algunos usan [DFS](https://www.asus.com/es/support/faq/1045936/) para evitar interferir con radares.
* Canales más anchos dan más velocidad pero consumen más espectro y generan más solapamiento.

#### Canales con varios APs

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

Muchos APs vecinos en el mismo canal compiten por el mismo medio y el rendimiento baja para todos.

### Qué suele fallar

<table><thead><tr><th width="332.40625">Síntoma</th><th>Causa probable</th></tr></thead><tbody><tr><td>Velocidad baja y señal buena</td><td>Canal saturado o cliente conectado en 2,4 GHz</td></tr><tr><td>Corte al cambiar de zona</td><td>Roaming mal configurado entre APs</td></tr><tr><td>Dispositivo antiguo no conecta</td><td>Solo soporta 2,4 GHz o un estándar anterior</td></tr></tbody></table>
