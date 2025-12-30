# Nmap

## PARÁMETROS DE NMAP

{% stepper %}
{% step %}
#### sP (Sondeo de Ping)

Esto sirve para detectar equipos conectados a mi red utilizando un ping.
{% endstep %}

{% step %}
#### -sn

Este parámetro sirve para detectar equipos encendidos dentro de nuestra web, es algo similar al arp-scan:

```
pyrhos@debian-laptop:~$ nmap -sn 192.168.1.1/24 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-20 09:01 CET
Nmap scan report for router.asus.com (192.168.1.1)
Host is up (0.013s latency).
Nmap scan report for debian-laptop (192.168.1.233)
Host is up (0.00072s latency).
Nmap scan report for jaime-aspirea31542 (192.168.1.245)
Host is up (0.11s latency).
Nmap scan report for 192.168.1.249
Host is up (0.096s latency).
Nmap scan report for HONOR-200-Lite (192.168.1.251)
Host is up (0.13s latency).
Nmap done: 256 IP addresses (5 hosts up) scanned in 7.12 seconds
```
{% endstep %}

{% step %}
#### -sS (TCP Syn)

Con esta opción podremos hacer un escaneo más sigiloso sin dejar rastro en la máquina objetivo:

```
  -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
```
{% endstep %}

{% step %}
#### -T (Plantillas de temporizado)

El parámetro -T de nmap sirve para aplicar una plantilla para hacer más rápidos o no los escaneos; y tenemos hasta 5 posibles plantillas, siendo la 5 la más rápida.

```
  -T<0-5>: Set timing template (higher is faster)
```
{% endstep %}

{% step %}
#### -vvv (Triple verbose)

Esto sirve para que a medida que encuentre algo, nos lo vaya reportando al mismo tiempo:

```
pyrhos@debian-laptop:~$ nmap 192.168.1.1 -vvv
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-20 09:05 CET
Initiating Ping Scan at 09:05
Scanning 192.168.1.1 [2 ports]
Completed Ping Scan at 09:05, 0.02s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 09:05
Completed Parallel DNS resolution of 1 host. at 09:05, 0.01s elapsed
DNS resolution of 1 IPs took 0.01s. Mode: Async [#: 1, OK: 1, NX: 0, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 09:05
Scanning router.asus.com (192.168.1.1) [1000 ports]
Discovered open port 53/tcp on 192.168.1.1
Discovered open port 80/tcp on 192.168.1.1
Discovered open port 9998/tcp on 192.168.1.1
Discovered open port 515/tcp on 192.168.1.1
Discovered open port 9100/tcp on 192.168.1.1
Completed Connect Scan at 09:05, 0.14s elapsed (1000 total ports)
Nmap scan report for router.asus.com (192.168.1.1)
Host is up, received syn-ack (0.0071s latency).
Scanned at 2025-11-20 09:05:50 CET for 0s
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE SERVICE    REASON
53/tcp   open  domain     syn-ack
80/tcp   open  http       syn-ack
515/tcp  open  printer    syn-ack
9100/tcp open  jetdirect  syn-ack
9998/tcp open  distinct32 syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.20 seconds

```
{% endstep %}

{% step %}
#### Rango de puertos (-p)

Con el parámetro -p podemos elegir que puertos escanear o definir un rango de puertos que escanee.

Escaneo de un puerto o puertos en especifico

```
nmap -p22 <IP>
nmap -p22,80,443 <IP>
```

Escaneo de un rango de puertos:

```
nmap -p1-65565 <IP>
```

O también se puede poner de estas manera:

```
nmap -p- <IP>
```

#### Sondeo UDP (-sU)

Para hacer escaneos por UDP utilizamos la opción -sU:

```
pyrhos@debian-laptop:~$ sudo nmap -sU 192.168.1.1 --top-ports 100
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-20 09:16 CET
Nmap scan report for router.asus.com (192.168.1.1)
Host is up (0.018s latency).
Not shown: 96 closed udp ports (port-unreach)
PORT     STATE SERVICE
53/udp   open  domain
67/udp   open  dhcps
1900/udp open  upnp
5353/udp open  zeroconf
MAC Address: 2C:4D:54:1A:E8:10 (ASUSTek Computer)

Nmap done: 1 IP address (1 host up) scanned in 94.67 seconds

```
{% endstep %}

{% step %}
#### Sondeo TCP ACK (Detectar tipo de Firewall)

Con la opcion -sA podemos detectar el tipo de firewall:

```
nmap -sA <ip>
```
{% endstep %}

{% step %}
#### Scripts de Nmap (resumen)

* Con <kbd>nmap --scrip default</kbd> ejecutamos los scripts de nmap por defecto para obtener información de las máquinas objetivo:
* El script <kbd>nmap --script "vuln and safe"</kbd> brinda información sobre vulnerabilidades:
* El script <kbd>nmap --script all</kbd> ejecuta todos los scripts de nmap:
* El script <kbd>nmap --script auth</kbd> comprueba contraseñas vacías o por defecto:
* <kbd>nmap --script=http-enum</kbd> fuzzer / enumerador web
* Fuerza bruta FTP (ejemplo con script de nmap)

`nmap --script=ftp-brute --script=args userdb="users.txt",passdb="pass.txt" -p21 "ip"`

* Fuerza bruta SSH (ssh-brute):

`nmap --script=ssh-brute --script=args userdb="users.txt",passdb="pass.txt" -p22 "ip"`

* Detección de vulnerabilidades en firewall (ejemplo):

`nmap -n -Pn -v --script=firewall-bypass --script args firewall-bypass,helper="ftp",firewall-bypass,target=22 <ip>`
{% endstep %}

{% step %}
#### -f (fragmentar paquetes)

Con el parámetro -f podemos fragmentar los paquetes a la hora de hacer el escaneo, por lo que de esta forma es posible que los puertos que se encuentren filtrados podamos llegar a verlos:
{% endstep %}

{% step %}
#### --script-mpu

El parámetro mpu sirve para ajustar el tamaño máximo de paquete sin fragmentar (MPU) para los paquetes enviados durante las pruebas de detección de firewall. Debe ponerse siempre un número múltiplo de 8 (8, 16, ...):
{% endstep %}

{% step %}
#### -D (decoy)

Sirve para confundir el firewall indicando direcciones IP señuelo, disimulando el análisis:

Si guardamos esto en un fichero .cap y lo analizamos con Wireshark veremos tráfico también desde la IP señuelo:
{% endstep %}

{% step %}
#### --spoof-mac (cambiar MAC)

Con la opción --spoof-mac podremos cambiar la MAC
{% endstep %}
{% endstepper %}

***

#### GUARDAR TRÁFICO DE NMAP EN FICHERO .CAP CON TCPDUMP

Podemos utilizar tcpdump para guardar un determinado tráfico que corre por una interfaz dentro de un fichero con extensión .cap.

Primero nos ponemos en escucha con tcpdump guardando el tráfico en un fichero y por la interfaz correspondiente:

Mientras tcpdump está en escucha, lanzamos por ejemplo un escaneo al puerto 22:

!\[\[Pasted image 20230412012708.png]]

Se hace el reporte:

!\[\[Pasted image 20230412012729.png]]

Ya tenemos el fichero .cap con el tráfico interceptado:

!\[\[Pasted image 20230412012801.png]]

Ahora lanzamos Wireshark con la captura:

!\[\[Pasted image 20230412012839.png]] !\[\[Pasted image 20230412012855.png]]

***

#### ESCANEOS SILENCIOSOS CON NMAP

Para hacer un escaneo lo más silencioso posible, puede usar:

* -sS para SYN Stealth (en lugar de TCP completo).
* -T2 para velocidad de escaneo baja.
* -Pn para no hacer detección de hosts.

Ejemplo:

```python
nmap -sS -T2 -Pn <dirección_ip>
```

Donde:

* "-sS" escaneo SYN Stealth.
* "-T2" velocidad baja.
* "-Pn" desactiva detección de hosts (no ping).

***

#### NMAP EN WINDOWS (Con Zenmap)

Zenmap es el asistente gráfico de nmap en Windows:

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

También se pueden crear y usar perfiles con objetivos y parámetros guardados:

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

***

#### SOURCE PORT (Regular el origen de los paquetes)

Por defecto nmap usa un puerto de origen aleatorio. Algunos firewalls permiten puertos comunes en lista blanca; se puede forzar el puerto de origen (ej. 53):



***

#### DATA LENGTH (Modificar la longitud del paquete)

Algunos firewalls filtran según tamaño de paquete. Podemos ajustar el mínimo y añadir longitud adicional con --data-length:

!\[\[Pasted image 20230413075148.png]] !\[\[Pasted image 20230413075434.png]]

En Wireshark se verá el aumento del tamaño de los paquetes:

!\[\[Pasted image 20230413075730.png]]

***

## SCRIPTS DE NMAP

Nmap cuenta con scripts para automatizar reconocimiento. Ejemplos:

* -sC: ejecutar scripts por defecto; combinado con -sV muestra versiones de servicios:

!\[\[Pasted image 20230413081220.png]]

* Ejecutar un script concreto (ej. vuln):

!\[\[Pasted image 20230413081456.png]]

* Ejecutar vuln junto a safe (menos intrusivo que vuln solo):

!\[\[Pasted image 20230413081949.png]]

* http-enum para fuzzing web sencillo:

!\[\[Pasted image 20230413082307.png]]

***

#### CÓMO HACER UN ESCANEO DE NMAP MUCHO MÁS RÁPIDO Y EFICIENTE

* Ponemos triple verbose para ver resultados mientras analiza.
* -n evita la resolución DNS.
* \--min-rate acelera.
* -Pn evita comprobación de ping previa.
* -oN / -oG para exportar resultados.

Ejemplo:

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 10.10.11.191 -oN escaneo
```

***
