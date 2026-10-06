# Nivel Físico

## Introducción

> La **capa física** es la encargada de convertir los bits en señales capaces de viajar por un medio de transmisión. Comprender cómo funcionan las señales eléctricas, ópticas e inalámbricas permite entender qué ocurre realmente cuando los datos circulan por un cable de red, una fibra óptica o una conexión Wi-Fi.

La información se transmite mediante una **señal** adaptada al medio empleado.

\
**Podemos distinguir principalmente tres tipos de señales:**

* **Eléctricas:** representan los datos mediante variaciones eléctricas que circulan por **cables de cobre**, como ocurre en determinadas conexiones Ethernet.
* **Ópticas:** representan los datos mediante **pulsos de luz** transmitidos a través de fibra óptica.
* **Inalámbricas:** utilizan **ondas electromagnéticas**, como ondas de radio o microondas, que se propagan por el espacio. Es el caso de tecnologías como **Wi-Fi**.

**Una señal puede describirse mediante diferentes características:**

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

* **Amplitud:** indica la intensidad o valor de la señal en un momento determinado.
* **Frecuencia:** representa el número de ciclos que se producen por segundo y se mide en **hercios (Hz)**. Por ejemplo, las redes Wi-Fi utilizan bandas como **2,4 GHz, 5 GHz o 6 GHz**.
* **Fase:** indica la posición de una señal dentro de su ciclo respecto a una referencia temporal.

Para transportar información puede utilizarse la **modulación**, que consiste en modificar determinadas características de una señal portadora para representar los datos que se desean transmitir.

Un ejemplo son los adaptadores **PLC (**_**Power Line Communications**_**)**, que permiten transportar datos utilizando el cableado de la red eléctrica.

Según la forma en la que varían, podemos distinguir:

* **Señales digitales:** utilizan un conjunto de valores discretos para representar la información.
* **Señales analógicas:** pueden variar de forma continua en el tiempo y en amplitud.

La **capa física** se encarga de convertir los bits en las señales adecuadas al medio utilizado: eléctricas, ópticas o electromagnéticas. En el dispositivo receptor realiza el proceso inverso, interpretando las señales recibidas para recuperar nuevamente los **bits originales**.

## Normas y asociaciones

> Para que dispositivos, redes y servicios de distintos fabricantes puedan funcionar juntos necesitan **normas comunes**. Conocer los principales organismos de estandarización permite entender de dónde proceden tecnologías tan habituales como **Ethernet, Wi-Fi, TCP/IP, GSM o las normas UNE**, y por qué son compatibles entre sí.

**Los estándares pueden clasificarse en dos tipos:**

1. **De facto o de hecho**: se convierten en estándar debido a su amplia utilización y aceptación. Un ejemplo clásico es la distribución de teclado **QWERTY**.&#x20;
2. **De iure o de derecho**: son aprobados formalmente por **organismos de normalización**. Un ejemplo es el modelo **OSI.**

**Los principales organismos relacionados con las telecomunicaciones y las redes son:**

<table><thead><tr><th width="108.1953125"></th><th></th></tr></thead><tbody><tr><td><strong>ITU</strong></td><td>organismo especializado de las <strong>Naciones Unidas</strong> encargado de las telecomunicaciones y las tecnologías digitales a escala internacional. Se estructura  en <strong>ITU-R</strong> (radiocomunicaciones), <strong>ITU-T</strong> (normalización) e <strong>ITU-D</strong> (desarrollo)</td></tr><tr><td><strong>ISO</strong></td><td>organización internacional de normalización formada por organismos nacionales de numerosos países. Entre sus trabajos relacionados con redes destaca el <strong>modelo OS</strong></td></tr><tr><td><strong>ANSI</strong></td><td>coordina y acredita estándares en <strong>Estados Unidos</strong> y representa al país en diferentes organismos internacionales de normalización.</td></tr><tr><td><strong>IEEE</strong></td><td>desarrolla numerosos estándares técnicos. En redes destacan los de la familia <strong>IEEE 802</strong>, como <strong>802.3 para Ethernet</strong> y <strong>802.11 para Wi-Fi</strong>.</td></tr><tr><td><strong>ETSI</strong></td><td>desarrolla estándares de telecomunicaciones y tecnologías digitales en Europa. Ha participado en tecnologías como <strong>GSM, 4G y 5G</strong>.</td></tr><tr><td><strong>UNE</strong></td><td>es el organismo español de normalización encargado actualmente de desarrollar y publicar las <strong>normas UNE</strong>. AENOR continúa realizando principalmente actividades de certificación y servicios relacionados</td></tr><tr><td><strong>ISOC</strong></td><td>promueve el desarrollo abierto de Internet y apoya el trabajo de organizaciones técnicas relacionadas con su evolución.</td></tr><tr><td><strong>IETF</strong></td><td>desarrolla y mantiene muchos de los estándares técnicos utilizados en Internet, publicados principalmente mediante documentos <strong>RFC</strong>.</td></tr><tr><td><strong>ICANN</strong></td><td>coordina elementos esenciales del sistema de identificadores de Internet, especialmente el <strong>sistema de nombres de dominio</strong> y determinados recursos relacionados con direcciones y parámetros de protocolos.</td></tr></tbody></table>

