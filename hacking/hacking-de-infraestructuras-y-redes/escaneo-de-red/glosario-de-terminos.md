# Glosario de Términos

## TTY

Una TTY en Linux es una interfaz que te permite interactuar con el sistema operativo a través de texto. Es decir, la TTY es la consola básica y directa de Linux, accesible sin entorno gráfico.

## Terminal

Una terminal, en cambio, es un programa que emula esa consola dentro de un entorno gráfico, como GNOME Terminal o Konsole.

## Pkexec

Es una herramienta en Linux que permite a los usuarios ejecutar programas con privilegios elevados (como si fueran el usuario root) de manera segura. Es parte del paquete PolicyKit (polkit), que gestiona permisos para ejecutar tareas administrativas en el sistema.

## EDR

En el contexto de la ciberseguridad, EDR significa Endpoint Detection and Response (Detección y Respuesta en el Endpoint). Es una solución de seguridad que monitorea continuamente los dispositivos finales (endpoints) de una red, como computadoras, servidores y dispositivos móviles, para detectar y responder a amenazas cibernéticas. EDR es una tecnología crucial para proteger los dispositivos en una red y para la detección proactiva y respuesta a amenazas cibernéticas avanzadas.

## XDR

Significa Extended Detection and Response (Detección y Respuesta Extendida). Es una evolución del concepto de EDR que amplía la detección y respuesta más allá de los endpoints para incluir múltiples capas de la infraestructura de TI, como redes, servidores, aplicaciones y cargas de trabajo en la nube. Mientras que EDR se centra principalmente en la protección de los endpoints, XDR integra la detección y respuesta a través de varias capas del entorno de TI, proporcionando una visión más completa de la seguridad.

## PARTICIÓN

Una partición de un disco duro es una división lógica del espacio de almacenamiento en un disco físico. Imagina un disco duro como un gran contenedor de datos; al crear particiones, estás dividiendo ese contenedor en compartimentos más pequeños, cada uno con su propio sistema de archivos y estructura de datos.

Por ejemplo, podemos ver como sda o sdb son los discos duros y sda1 y sdb1 son particiones:\
!\[\[Pasted image 20240828150449.png]]

## SISTEMA DE FICHEROS

El sistema de ficheros, también conocido como sistema de archivos, es la forma en que un sistema operativo organiza y gestiona los datos en un disco duro o en otro dispositivo de almacenamiento. Existen varios tipos de sistemas de ficheros, y cada uno tiene características y ventajas específicas. Algunos ejemplos incluyen:

* FAT32: Un sistema de archivos más antiguo y simple, compatible con una amplia gama de dispositivos y sistemas operativos, pero con limitaciones en cuanto a tamaño de archivos y volumen.
* NTFS: Utilizado principalmente en sistemas Windows, soporta archivos grandes, ofrece mejor seguridad y tiene características avanzadas como la compresión y el cifrado.
* ext4: Común en sistemas Linux, ofrece buen rendimiento, fiabilidad y características avanzadas como el soporte para grandes volúmenes y archivos.

## CIFRAR

Cifrar se refiere al proceso de convertir datos o información en un formato que solo puede ser leído o interpretado por personas o sistemas autorizados. Es decir, cuando un disco está cifrado, el contenido se vuelve inaccesible sin la clave correcta.

## ENCRIPTAR

Encriptar significa convertir información o datos de un formato legible en uno ilegible mediante un algoritmo criptográfico. Este proceso usa una clave para transformar el texto original (texto en claro) en un formato cifrado que solo puede ser leído de nuevo mediante un proceso inverso, conocido como descifrado, si se tiene la clave adecuada.

## BASTIONADO

En el contexto de la informática y servidores, el término "bastionado" (o "bastionado") se refiere a un enfoque de seguridad en el que se utiliza un servidor o una máquina, conocida como "servidor bastión" o "bastion host", para proteger y controlar el acceso a una red interna.
