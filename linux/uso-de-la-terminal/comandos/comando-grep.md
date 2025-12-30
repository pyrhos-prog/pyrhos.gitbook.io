# Comando grep

Con este comando podemos hacer búsquedas, encontrando líneas donde se encuentra el patrón que estamos buscando. Por ejemplo, vamos a buscar el nombre `Pyrhos` dentro del fichero `/etc/passwd`:

{% stepper %}
{% step %}
#### Búsqueda en un fichero

Comando usado:

```bash
grep <usuario> /etc/passwd
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Buscar alternativas con expresión regular

También podemos buscar donde queramos encontrar una cosa o bien otra; para ello utilizamos la opción `-E` .

```bash
grep -E "<palabra>|<palabra>" <ruta>
```

Para encontrar `pyrhos` o `usbmux`:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Buscar en varios ficheros

Podemos pasarle el nombre del archivo o usar un asterisco para buscar en varios archivos. Por ejemplo, buscar todas las coincidencias de un nombre en todos los archivos del escritorio:

Comando usado:

```bash
grep pyrhos *
```
{% endstep %}

{% step %}
#### Suprimir mensajes de error

Si quieres quitar los mensajes de error (por ejemplo cuando grep intenta leer directorios), usa la opción `-s`:

Comando usado:

```bash
grep -s pyrhos *
```
{% endstep %}

{% step %}
#### Contar cuántas líneas repite un patrón (-c)

Podemos saber en cuántas líneas se repite un patrón usando la opción `-c`. Por ejemplo, si queremos saber cuántas líneas contienen `ttl=115` dentro del fichero `ping.log`:

```bash
grep -c "ttl=64" ping.log
```

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
#### Eliminar un patrón con grep (-v)

Con el parametro -v grep elimina una palabra que indiquemos del output.

```bash
grep -v body archivo.txt
```
{% endstep %}
{% endstepper %}
