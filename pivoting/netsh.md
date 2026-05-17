# Netsh

Netsh es una herramienta de windows que sirve para la configuracion de red de windows de un sistema en particular. Esta herramienta permite:

* Encontrar rutas
* Ver la configuración del firewall
* Añadir proxies
* Crear reglas de port forwarding

**Para entenderlo mejor vamos a usar el siguiente escenario:**

El host comprometido es el ordenador de un administrador IT con windows 10 (10.129.15.150, 172.16.5.19).

<figure><img src="../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>



## Reenviar puertos con Netsh

### 1. Configurar portproxy

```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```

### 2. Verificación del reenvío del puerto

```cmd
netsh.exe interface portproxy show v4tov4
```

Nos debería de devolver algo así:

```cmd
Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.15.150   8080        172.16.5.25     3389
```

### 3. Conexión desde la maquina atacante

Una vez el port forwarding este configurado podemos conectarnos desde nuestra maquina atacante, en este caso hemos configurado el puerto 3389 que es el puerto por defecto de RDP

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>
