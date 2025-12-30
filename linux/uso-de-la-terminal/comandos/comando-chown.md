# comando chown

{% stepper %}
{% step %}
### Cambiar el propietario de un archivo

El comando `chown` se utiliza para cambiar el propietario de un archivo o directorio.

```bash
sudo chown usuario archivo
o
sudo chown usuario:grupo archivo
```
{% endstep %}

{% step %}
### Notación de propietario y grupo

El nombre de la izquierda es el usuario propietario y el nombre de la derecha es el grupo:

```bash
-rw-r--r-- 1 usuario grupo  25 Feb 16 15:30 archivo.txt
```
{% endstep %}
{% endstepper %}

{% hint style="info" %}
En la salida de `ls -l`, el formato es:

* Permisos
* Número de enlaces
* Usuario (propietario)
* Grupo
* Tamaño, fecha y nombre del archivo
{% endhint %}
