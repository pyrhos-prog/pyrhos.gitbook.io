# Protocolos de seguridad

## Seguridad Inalámbrica

La seguridad inalámbrica funciona mediante el uso de contraseñas y cifrado para proteger la conexión a internet. hay diferentes tipos de cifrados, Como el WEP, WPA2 o WPA3,&#x20;



## Protocolos de seguridad Wi-Fi

Los protocolos funcionan implementando medidas de seguridad como el cifrado y la autentificación.



**Cifrado**

Hace que la comunicación sea ininteligible para cualquiera menos para los que tengan las claves de cifrado.

**Autenticación**

Permite que solo los dispositivos con identidades verificadas puedan unirse a la red.



### Protocolo WEP

El protocolo WEP fue implementado en 1999, es el mas antiguo y vulnerable

#### Características&#x20;

* Usa cifrado RC4
* Esta obsoleto y es inseguro
* Es vulnerable a ataques de fuerza bruta



### Protocolo WPA

El protocolo WPA tomó el relevo a WEP, fué una mejora temporal&#x20;

#### Caraterísticas

* Utiliza claves dinámicas
* Compatible con hardware antiguo
* Utiliza TKIP sobre RC4



### Protocolo WPA2

El protocolo WPA2 se introdujo en 2004 y su mejora mas importante fue el uso de la encriptación AES.

#### Caraterísticas

* Seguridad fuerte con contraseñas robustas
* Encriptación AES  (Advanced Encryption Standart)
* Es vulnerable a ataques WPS

**WPA2-PSK (Personal)**

* Autenticación mediante contraseña compartida.
* Vulnerable a ataques de diccionario si la clave es débil.
* Dependiente del handshake de 4 vías.

**WPA2-Enterprise**

* Usa servidor [RADIUS](https://www.redeszone.net/tutoriales/servidores/servidor-radius-cloud-autenticacion-clientes/).
* Autenticación individual por usuario.
* Más seguro en entornos corporativos.



### Protocolo WPA3

WPA3 es la nueva generación de seguridad Wi-Fi, con mejoras para la seguridad Wi-Fi.

#### Características

* Es resistente a ataques de fuerza bruta offline
* Tiene un cifrado mas robusto
* Seguridad individualizada para cada dispositivo conectado

{% embed url="https://www.incibe.es/ciudadania/blog/wpa3-el-protocolo-de-seguridad-configurar-en-tu-router" %}

