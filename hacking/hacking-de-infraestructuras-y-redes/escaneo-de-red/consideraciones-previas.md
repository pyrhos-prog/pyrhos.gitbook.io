# Consideraciones Previas

## COMPARTIR ARCHIVOS RECURSO COMPARTIDO CUANDO NOS DA UN ERROR DE CREDENCIALES

Vamos a compartir estos dos archivos creando un recurso compartido con impacket-smbserver de esta forma:

!\[\[Pasted image 20221217114648.png]]

{% stepper %}
{% step %}
### Crear el recurso compartido (sin credenciales)

Al intentar descargar los archivos desde la máquina víctima nos indica que debe haber un usuario y contraseña:

!\[\[Pasted image 20221217114654.png]]
{% endstep %}

{% step %}
### Crear el recurso compartido con usuario y contraseña

Creamos otra vez el recurso compartido fijando un usuario y contraseña:

!\[\[Pasted image 20221217114701.png]]
{% endstep %}

{% step %}
### Descargar todo el recurso compartido

Ahora descargamos todo el recurso compartido de esta forma:

!\[\[Pasted image 20221217114707.png]]
{% endstep %}

{% step %}
### Acceder a los archivos desde la unidad montada

Si hacemos un dir de la unidad (x es la unidad creada donde se almacena todo el recurso compartido), accedemos a todos los archivos:

!\[\[Pasted image 20221217114715.png]]
{% endstep %}
{% endstepper %}

## EJECUTAR NETCAT EN WINDOWS

Para ejecutar netcat en Windows debemos tener el ejecutable de netcat descargado y ejecutamos el siguiente comando:

!\[\[Pasted image 20221217114732.png]]

Puede darse el caso donde tengamos que poner primero un ./ antes de ejecutar netcat:

!\[\[Pasted image 20221217114739.png]]

Aunque también puede darse el caso de que tengamos que poner un ./ antes del ejecutable para ejecutarlo.

### CONEXIÓN POR TELNET

Para conectarse por Telnet, en el equipo servidor debe tener configurado correctamente este servicio. Una vez lo tenga configurado, ya podemos iniciar sesión utilizando el nombre del login del equipo de destino y su contraseña de iniciar sesión, escribiendo:

```bash
telnet <dirección IP>
```

## TRATAMIENTO DE LA TTY

Para crearnos una pseudo-consola donde podamos hacer Control-C y no tengamos problemas con el teclado ejecutamos lo siguiente:

```bash
script /dev/null -c bash
control z
stty raw -echo; fg
reset xterm
export SHELL=bash
export TERM=xterm
```

!\[\[Pasted image 20230502104304.png]] !\[\[Pasted image 20230502104222.png]]

### ENCONTRAR DIRECCIÓN IPV6 EN MÁQUINA LINUX

Hay una ruta donde podemos encontrar la dirección IPv6 de una máquina Linux, y es la siguiente:

```bash
cat /proc/net/if_inet6
```

!\[\[Pasted image 20240105111122.png]]

Esto puede ser muy útil cuando queramos obtener dicha dirección tras explotar un LFI:

```bash
curl -sX GET 'http://192.168.0.32/php-scripts/file.php?6=/proc/net/if_inet6'
```

!\[\[Pasted image 20240105111152.png]]

No obstante, por aquí todavía no encontramos la IPv6; y para ello podemos utilizar ping6 de la siguiente forma:

```bash
ping6 -c2 -n -I eth0 ff02::1
```

{% hint style="info" %}
Explicación de los parámetros utilizados:

* `ping6`: Envía paquetes ICMPv6 de solicitud de eco.
* `-c2`: Envía 2 paquetes antes de detenerse.
* `-I eth0`: Especifica la interfaz de red (aquí `eth0`).
* `ff02::1`: Dirección de multidifusión de enlace local para todos los nodos.
{% endhint %}

Al ejecutarlo nos devuelve muchas direcciones IPv6:

!\[\[Pasted image 20240105112339.png]]

Vamos probando a conectarnos por SSH con cada una de estas IPs hasta acertar:

```bash
ssh -6 -i id_rsa cromiphi@fe80::a00:27ff:fe4c:9dd4%eth0
```

Utilizando el archivo id\_rsa con la contraseña ilovemyself del fichero id\_rsa:

!\[\[Pasted image 20240105112413.png]]
