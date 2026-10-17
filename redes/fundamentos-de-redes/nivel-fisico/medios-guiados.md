# Medios guiados

Medios en los que la señal viaja por un soporte físico: coaxial, par trenzado, cable eléctrico (PLC) y fibra óptica.

### Sentido de la comunicación

| Modo        | Descripción                      | Ejemplo                                        |
| ----------- | -------------------------------- | ---------------------------------------------- |
| Simplex     | Un solo sentido                  | Radio o TV tradicional                         |
| Half duplex | Ambos sentidos, pero no a la vez | Walkie-talkie, Ethernet con hubs               |
| Full duplex | Ambos sentidos a la vez          | Ethernet conmutada moderna, llamada telefónica |

### Comparativa rápida

| Medio        | Señal                                    | Dónde se usa                                                 | Pros                                                                  | Contras                                         |
| ------------ | ---------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------------- | ----------------------------------------------- |
| Coaxial      | Eléctrica de alta frecuencia             | TV por cable (DOCSIS), videovigilancia, RF, Ethernet antigua | Buen apantallamiento                                                  | Sustituido en redes de datos                    |
| Par trenzado | Eléctrica                                | Casi toda LAN actual                                         | Barato, flexible                                                      | Límite de 100 m, sensible a ruido               |
| PLC          | Datos sobre red eléctrica (230 V, 50 Hz) | Llevar red a una habitación sin Wi-Fi                        | No requiere cableado nuevo                                            | Rendimiento depende de la instalación eléctrica |
| Fibra óptica | Pulsos de luz                            | Backbone, enlaces largos, CPD, FTTH                          | Gran ancho de banda, inmune a interferencias EM, no conduce corriente | Herramientas y conectores más caros             |

### Par trenzado

Cuatro pares (ocho hilos) con conector RJ45.

#### Apantallamiento

| Código | Significado                       | Cuándo                              |
| ------ | --------------------------------- | ----------------------------------- |
| U/UTP  | Sin pantalla                      | Oficina estándar, barato y flexible |
| F/UTP  | Pantalla global                   | Entornos con más interferencias     |
| S/FTP  | Malla exterior y pantalla por par | Máxima protección frente a ruido    |

#### Categorías

| Categoría | Velocidad de referencia                          | Uso                                    |
| --------- | ------------------------------------------------ | -------------------------------------- |
| 5e        | Hasta 1 Gbit/s                                   | Redes de 1 Gbit/s                      |
| 6         | 1 Gbit/s a 100 m; 10 Gbit/s a distancias menores | Oficina actual                         |
| 6A        | 10 Gbit/s a 100 m                                | Instalación nueva con margen de futuro |
| 8         | 25 y 40 Gbit/s en enlaces cortos                 | Centros de datos                       |

Decisión práctica: para 10 Gbit/s a unos 100 m, Cat 6A; Cat 5e se queda corta.

### Fibra óptica

| Tipo                     | Núcleo / revestimiento        | Luz              | Uso                             | Color típico                      |
| ------------------------ | ----------------------------- | ---------------- | ------------------------------- | --------------------------------- |
| Monomodo (OS2)           | \~9 µm / 125 µm               | Un único camino  | Larga distancia, alta velocidad | Amarillo                          |
| Multimodo                | 50 o 62,5 µm / 125 µm         | Varios caminos   | Edificios y CPD                 | Naranja (OM1/OM2), aqua (OM3/OM4) |
| Multimodo índice gradual | Índice de refracción variable | Menor dispersión | Mejora la transmisión multimodo | Según norma                       |

Los colores ayudan a identificar el tipo, pero pueden variar según fabricante y norma. Se comprueba siempre la serigrafía de la cubierta.

Regla de decisión:

* Distancia grande, interferencias electromagnéticas o mucho ancho de banda: fibra.
* Puestos de usuario a menos de 100 m: par trenzado.

### Qué se rompe

| Síntoma                              | Causa probable                                                           |
| ------------------------------------ | ------------------------------------------------------------------------ |
| Enlace negocia a 100 Mbit/s          | Par defectuoso o categoría insuficiente                                  |
| Errores CRC o pérdidas intermitentes | Ruido, diafonía o conector mal crimpado                                  |
| Fibra sin enlace                     | Conectores sucios, TX/RX invertidos, tipo de fibra o módulo incompatible |
