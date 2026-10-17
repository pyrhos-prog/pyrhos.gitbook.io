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

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

* **Amplitud:** indica la intensidad o valor de la señal en un momento determinado.
* **Frecuencia:** representa el número de ciclos que se producen por segundo y se mide en **hercios (Hz)**. Por ejemplo, las redes Wi-Fi utilizan bandas como **2,4 GHz, 5 GHz o 6 GHz**.
* **Fase:** indica la posición de una señal dentro de su ciclo respecto a una referencia temporal.

Para transportar información puede utilizarse la **modulación**, que consiste en modificar determinadas características de una señal portadora para representar los datos que se desean transmitir.

Un ejemplo son los adaptadores **PLC (**_**Power Line Communications**_**)**, que permiten transportar datos utilizando el cableado de la red eléctrica.

Según la forma en la que varían, podemos distinguir:

* **Señales digitales:** utilizan un conjunto de valores discretos para representar la información.
* **Señales analógicas:** pueden variar de forma continua en el tiempo y en amplitud.

La **capa física** se encarga de convertir los bits en las señales adecuadas al medio utilizado: eléctricas, ópticas o electromagnéticas. En el dispositivo receptor realiza el proceso inverso, interpretando las señales recibidas para recuperar nuevamente los **bits originales**.
