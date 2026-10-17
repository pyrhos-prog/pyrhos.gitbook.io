# Medios inalámbricos

> La señal viaja por el espacio como onda electromagnética. Frecuencia y longitud de onda son inversamente proporcionales, y de la frecuencia depende casi todo: alcance, capacidad y capacidad de atravesar obstáculos.

### Regla de las frecuencias

| Frecuencia | Alcance y obstáculos         | Direccionalidad | Capacidad |
| ---------- | ---------------------------- | --------------- | --------- |
| Más baja   | Cubre más y atraviesa mejor  | Menor           | Menor     |
| Más alta   | Cubre menos y atraviesa peor | Mayor           | Mayor     |

Por eso 2,4 GHz llega más lejos que 5 GHz, y 5 GHz va más rápido.

### Qué medio elegir

| Medio                   | Características                                                                                       | Casos de uso                                              |
| ----------------------- | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| Ondas de radio          | Omnidireccionales, cobertura amplia. Bandas LF, MF, HF, VHF, UHF                                      | AM (MF), FM (VHF), TV, móvil y Wi-Fi (UHF y microondas)   |
| Microondas terrestres   | Antenas direccionales, requieren línea de visión; la lluvia aumenta la atenuación a frecuencias altas | Enlace punto a punto entre edificios o estaciones base    |
| Microondas por satélite | Enlace ascendente y descendente en bandas distintas                                                   | TV por satélite, navegación, Internet por satélite        |
| Infrarrojos             | Corto alcance, visión directa, no atraviesan paredes                                                  | Mandos a distancia, sensores, termografía                 |
| Luz / láser             | Muy direccional, alineación precisa                                                                   | Enlaces ópticos en espacio libre entre edificios cercanos |

Wi-Fi, Bluetooth y Zigbee trabajan en bandas del rango de las microondas.

### Ejemplo: unir dos edificios con visión directa

Necesidad: enlace punto a punto, mucha capacidad, sin dar cobertura alrededor.

* La **opción correcta** sería usar un enlace de microondas direccional o láser, bien alineado.
* Las **opciones incorrectas** son usar una antena Wi-Fi omnidireccional (reparte señal donde no hace falta), infrarrojos (corto alcance y no atraviesan obstáculos), frecuencias bajas omnidireccionales (poca direccionalidad).

### Seguridad

El medio inalámbrico es compartido y cualquiera dentro de su alcance puede escuchar. Por eso el cifrado en la capa de enlace (WPA2/WPA3) no es opcional.
