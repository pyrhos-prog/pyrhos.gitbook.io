# Enumeracion inicial de dominio

| **Punto de datos**              | **Descripción**                                                                                                                                                                   |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AD Users`                      | Estamos tratando de enumerar las cuentas de usuario válidas a las que podemos dirigirnos para la pulverización de contraseñas.                                                    |
| `AD Joined Computers`           | Los equipos clave incluyen controladores de dominio, servidores de archivos, servidores SQL, servidores web, servidores de correo de Exchange, servidores de bases de datos, etc. |
| `Key Services`                  | Kerberos, NetBIOS, LDAP, DNS                                                                                                                                                      |
| `Vulnerable Hosts and Services` | Cualquier cosa que pueda ser una victoria rápida. (también conocido como un host fácil de explotar y afianzarse)                                                                  |

**TTPs**

Enumerar un entorno de AD puede ser abrumador si se aborda sin un plan. Hay una gran cantidad de datos almacenados en AD, y puede llevar mucho tiempo tamizarlos si no se miran en etapas progresivas, y es probable que nos perdamos cosas. Tenemos que establecer un plan de juego y abordarlo pieza por pieza. A medida que ganemos más experiencia, desarrollaremos nuestra propia metodología repetible. Independientemente de cómo procedamos, normalmente comenzamos en los mismos lugares y buscamos los mismos puntos de datos.

Comenzaremos con la identificación de los hosts de la red, seguiremos con la validación de los resultados para obtener más información sobre cada host (qué servicios se están ejecutando, nombres, posibles vulnerabilidades, etc.). Una vez que sabemos qué hosts existen, sondearemos esos hosts buscando cualquier dato interesante que podamos obtener. Después de esas tareas, nos reagruparmos y evaluamos la información recopilada. En este punto, es razonable esperar tener credenciales o una cuenta de usuario a la que apuntar para obtener un punto de apoyo en un host unido a dominio o bien la capacidad de comenzar la enumeración de credenciales desde nuestro host de ataque de Linux (`passive` `active`).

A continuación veremos herramientas y técnicas que nos ayudarán con esta enumeración.

{% stepper %}
{% step %}
### 1) Escuchar la red (identificación pasiva de hosts)

Tomarnos un tiempo para "poner el oído en el cable" y ver qué está pasando nos ayuda a encontrar hosts y tipos de tráfico sin enviar paquetes. Herramientas como Wireshark y tcpdump permiten capturar ARP, MDNS y otros paquetes de capa 2 (dado que estamos en una red conmutada, estamos limitados al dominio de difusión actual).

Ejemplo: iniciar Wireshark con privilegios sudo en el host de ataque.

```shell
sudo -E wireshark
```

Salida (parcial):

```
11:28:20.487     Main Warn QStandardPaths: runtime directory '/run/user/1001' is not owned by UID 0, but a directory permissions 0700 owned by UID 1001 GID 1002
<SNIP>
```

Ejemplo de captura (imagen):

![imagen](<../../.gitbook/assets/ea wireshark.png>)

Notas:

* Los paquetes ARP nos hacen conscientes de los hosts: 172.16.5.5, 172.16.5.25, 172.16.5.50, 172.16.5.100 y 172.16.5.125.
* MDNS nos informa sobre el host ACADEMY-EA-WEB01.

Si estamos en un host sin GUI podemos usar tcpdump, net-creds, NetMiner, etc., y guardar .pcap para analizarlo en otro host con Wireshark.
{% endstep %}

{% step %}
### 2) Escucha y análisis pasivo con Responder

Responder puede escuchar y analizar solicitudes LLMNR, NBT-NS y MDNS. Aquí lo usamos en modo Analizar (pasivo), que no envenena: solo escucha y muestra solicitudes/respuestas.

Iniciar Responder en interfaz ens224 en modo análisis:

```bash
sudo responder -I ens224 -A
```

Al iniciar Responder veremos solicitudes fluyendo en la sesión y podremos descubrir hosts adicionales no visibles en la captura anterior. Anotar los hosts detectados para enumeración posterior.
{% endstep %}

{% step %}
### 3) Barrido ICMP activo (fping)

Después de la recolección pasiva, podemos hacer verificaciones activas rápidas, por ejemplo con fping para determinar hosts vivos en un rango CIDR.

Ejemplo:

```bash
fping -asgq 172.16.5.0/23
```

Salida de ejemplo:

```
172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.200
172.16.5.225
172.16.5.238
172.16.5.240

     510 targets
       9 alive
     501 unreachable
       0 unknown addresses

    2004 timeouts (waiting for response)
    2013 ICMP Echos sent
       9 ICMP Echo Replies received
    2004 other ICMP received

 0.029 ms (min round trip time)
 0.396 ms (avg round trip time)
 0.799 ms (max round trip time)
       15.366 sec (elapsed real time)
```

fping es útil para sondear múltiples hosts de forma rotatoria y con capacidades de scripting.
{% endstep %}

{% step %}
### 4) Sondeo de servicios con Nmap (enumeración activa)

Con la lista de hosts activos, enumeramos servicios para identificar roles importantes (controladores de dominio, servidores web, correo, etc.) y para priorizar protocolos relevantes para AD (DNS, SMB, LDAP, Kerberos).

Ejemplo de escaneo Nmap usando una lista de hosts:

```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

Esto nos permitirá identificar servicios en cada host y detectar hosts críticos y potencialmente vulnerables para sondear más a fondo.
{% endstep %}
{% endstepper %}
