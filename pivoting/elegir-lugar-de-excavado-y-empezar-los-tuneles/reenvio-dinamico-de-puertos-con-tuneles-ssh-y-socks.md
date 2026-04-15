# Reenvío dinámico de puertos con túneles SSH y SOCKS

## Port forwarding&#x20;

Es una tecnica que permite redirigir solicitudes de comunicaciones de un puerto a otro. se utiliza TCP como capa de comunicacion principal pero tambien se pueden utilizar protocolos de capa de aplicacion como SSH o SOCKS para encapsular el tráfico reenviado.

&#x20;Utilizando protocolos de capa de aplicacion podemos:

* Eludir firewalls
* Utilizar servicios existentes en un host comprometido para cambiar a otras redes

### Reenvio de puertos con SSH

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

En este ejemplo tenemos nuestro host de ataque (10.10.15.x) y un servidor Ubuntu de destino (10.129.xx), que hemos comprometido.&#x20;

Escanearemos el servidor Ubuntu de destino usando Nmap para buscar puertos abiertos.

```bash
pyrhos@htb[/htb]$ nmap -sT -p22,3306 10.129.202.64

Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-24 12:12 EST
Nmap scan report for 10.129.202.64
Host is up (0.12s latency).

PORT     STATE  SERVICE
22/tcp   open   ssh
3306/tcp closed mysql

Nmap done: 1 IP address (1 host up) scanned in 0.68 seconds
```

* SSH (22) → accesible desde donde estás
* MySQL (3306) → **no accesible directamente desde fuera**

La base de datos MySQL está **solo en la red interna o bind a localhost,** para poder acceder hay que hacer un reenvio de puertos con SSH.

```bash
pyrhos@htb[/htb]$ ssh -L 1234:localhost:3306 ubuntu@10.129.202.64
# O para reenviar varios puertos
pyrhos@htb[/htb]$ ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```

<table><thead><tr><th width="97.90625">Parte</th><th>Significado</th></tr></thead><tbody><tr><td>-L</td><td> parametro que permite el port forwarding</td></tr><tr><td>1234</td><td>puerto de nuestra maquina</td></tr><tr><td>localhost</td><td>destino visto desde la maquina victima o maquina remota</td></tr><tr><td>3306</td><td>puerto de la maquina remota</td></tr></tbody></table>

Para comprobar que el port forwarding se ha hecho de manera correcta podemos usar el siguiente comando:

```bash
netstat -antp | grep 1234
```

### Configuración para pivoting

En este ejemplo el host de Ubuntu tiene mas de una interfaz de red:

* Uno conectado a nuestro host de ataque (`ens192`)
* Uno que se comunica con otros hosts dentro de una red diferente (`ens224`)
* La interfaz de loopback (`lo`).

