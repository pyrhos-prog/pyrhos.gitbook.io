# Handshake y cifrado

## Handshake

El handshake es el proceso mediante el que un cliente y un punto de acceso establecen una conexión. Durante este proceso se autentican y generan las claves necesarias para cifrar el tráfico.

### Handshake en WPA Y WPA2-PSK

Se utiliza el 4 Way Handshake

#### Objetivo

* Verificar que ambos conocen la contraseña
* Generar las claves temporales&#x20;
* Iniciar el cifrado de tráfico

#### Proceso del 4-Way Hanshake

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### Características

* El handshake puede capturarse
* Permite ataques de diccionario offline
* La seguridad depende de la contraseña

### Cifrado en WPA/WPA2-PSK

#### Algortimos

* **WPA:** utiliza TKIP, que es inseguro
* **WPA2-PSK**: utiliza AES, es seguro

{% embed url="https://borrowbits.com/2014/02/seguridad-en-wi-fi-que-cifrado-es-mejor-tkip-o-aes/" %}





### Handshake en WPA3

#### Objetivo

* Autenticación segura
* Evitar ataques offline
* Proteger la contraseña

#### Proceso del handshake en WPA3

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

#### Características

* No se puede capturar un handshake utilizable para ataques de diccionario offline
* Requiere interacción activa con el AP

### Cifrado en WPA3

#### Algoritmos

* Utiliza el AES CCMP O GCMP
* Las claves son mas fuertes
* Tiene una protección adicional contra ataque pasivos

