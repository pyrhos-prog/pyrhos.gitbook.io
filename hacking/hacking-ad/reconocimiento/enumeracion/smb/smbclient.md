# SMBclient

Smbclient es un cliente de línea de comandos para el protocolo de intercambio de archivos de red Server Message Block (SMB). Este protocolo se utiliza comúnmente para compartir archivos e impresoras entre dispositivos en una red local.

Smbclient permite a los usuarios conectarse a servidores SMB y acceder a recursos compartidos como si estuvieran en una unidad local de su computadora. Con smbclient, los usuarios pueden explorar recursos compartidos, descargar y cargar archivos, crear y eliminar directorios, y realizar otras operaciones de archivos a través de la red.

Smbclient es una herramienta muy útil para los administradores de redes y los usuarios avanzados que necesitan trabajar con archivos y recursos compartidos en una red de Windows. También es compatible con sistemas operativos basados en Unix y Linux.

### Ejemplos de uso

{% stepper %}
{% step %}
### Ejecutar comandos con smbclient

Por ejemplo, vamos a suponer que nos encontramos con estos recursos compartidos:

!\[\[Pasted image 20230213104857.png]]

Con smbclient podemos acceder dentro de un recurso compartido con las credenciales correspondientes:

!\[\[Pasted image 20230213105508.png]]
{% endstep %}

{% step %}
### Listar recursos compartidos con una null session

Podemos listar los recursos compartidos sin proporcionar credenciales haciendo uso de una null session:

!\[\[Pasted image 20230420110349.png]]
{% endstep %}

{% step %}
### Conectarnos a un recurso compartido SMB sin credenciales

Podríamos hacerlo simplemente especificando el nombre del recurso compartido al que nos queramos conectar:

!\[\[Pasted image 20230420110733.png]]
{% endstep %}

{% step %}
### Subir y descargar archivos

* Subir un archivo (comando put). Ejemplo: subir un documento txt:

!\[\[Pasted image 20230420110901.png]]

* Descargar un archivo (comando get). Ejemplo:

!\[\[Pasted image 20230420111148.png]]
{% endstep %}
{% endstepper %}

{% hint style="info" %}
Puede ser muy útil acceder con rpcclient al objetivo para obtener más información del sistema. \[\[Rpcclient y Enum4linux]]
{% endhint %}
