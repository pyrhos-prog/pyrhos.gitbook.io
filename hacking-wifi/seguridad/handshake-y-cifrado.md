---
icon: wifi
---

# Handshake y cifrado

### Handshake

El handshake es el proceso mediante el que un cliente y un punto de acceso establecen una conexión. Durante este proceso se autentican y generan las claves necesarias para cifrar el tráfico.

#### Handshake en WPA Y WPA2-PSK

Se utiliza el 4 Way Handshake

**Objetivo**

* Verificar que ambos conocen la contraseña
* Generar las claves temporales
* Iniciar el cifrado de tráfico

**Proceso del 4-Way Hanshake**

<figure><img src="https://3217721309-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FzWtmKFQPhEpxGVug9Wgi%2Fuploads%2FMAwNlnLtZI5CXKI5i3sQ%2Fimage.png?alt=media&#x26;token=af5b46e7-e6f2-46f5-9813-c1dd2927b787" alt=""><figcaption></figcaption></figure>

**Características**

* El handshake puede capturarse
* Permite ataques de diccionario offline
* La seguridad depende de la contraseña

#### Cifrado en WPA/WPA2-PSK

**Algortimos**

* **WPA:** utiliza TKIP, que es inseguro
* **WPA2-PSK**: utiliza AES, es seguro

#### Handshake en WPA3

**Objetivo**

* Autenticación segura
* Evitar ataques offline
* Proteger la contraseña

**Proceso del handshake en WPA3**

<figure><img src="https://3217721309-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FzWtmKFQPhEpxGVug9Wgi%2Fuploads%2FEI4lwxorVrnjZaAr2wWY%2Fimage.png?alt=media&#x26;token=31149c8e-da0f-4f45-a2ec-36e7f155d67f" alt=""><figcaption></figcaption></figure>

**Características**

* No se puede capturar un handshake utilizable para ataques de diccionario offline
* Requiere interacción activa con el AP

#### Cifrado en WPA3

**Algoritmos**

* Utiliza el AES CCMP O GCMP
* Las claves son mas fuertes
* Tiene una protección adicional contra ataque pasivos
