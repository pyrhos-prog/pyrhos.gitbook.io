---
icon: terminal
---

# Comando grep

Con este comando podemos hacer búsquedas, encontrando líneas donde se encuentra el patrón que estamos buscando. Por ejemplo, vamos a buscar el nombre `Pyrhos` dentro del fichero `/etc/passwd`:

{% stepper %}
{% step %}
**Búsqueda en un fichero**

Comando usado:

```bash
grep <usuario> /etc/passwd
```

<figure><img src="https://36118879-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F5YP0HIeDL3z0KLMojf1y%2Fuploads%2FeF7Z7WIw27GS2c9DjTYB%2Fimage.png?alt=media&#x26;token=71ebae48-62df-4c67-884d-2fe6a2eb797d" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**Buscar alternativas con expresión regular**

También podemos buscar donde queramos encontrar una cosa o bien otra; para ello utilizamos la opción `-E` .

```bash
grep -E "<palabra>|<palabra>" <ruta>
```

Para encontrar `pyrhos` o `usbmux`:

<figure><img src="https://36118879-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F5YP0HIeDL3z0KLMojf1y%2Fuploads%2F5DJw6fmelSAa7cXbF7Ov%2Fimage.png?alt=media&#x26;token=dfac923a-0273-4f13-af18-7113c7d01c6c" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**Buscar en varios ficheros**

Podemos pasarle el nombre del archivo o usar un asterisco para buscar en varios archivos. Por ejemplo, buscar todas las coincidencias de un nombre en todos los archivos del escritorio:

Comando usado:

```bash
grep pyrhos *
```
{% endstep %}

{% step %}
**Suprimir mensajes de error**

Si quieres quitar los mensajes de error (por ejemplo cuando grep intenta leer directorios), usa la opción `-s`:

Comando usado:

```bash
grep -s pyrhos *
```
{% endstep %}

{% step %}
**Contar cuántas líneas repite un patrón (-c)**

Podemos saber en cuántas líneas se repite un patrón usando la opción `-c`. Por ejemplo, si queremos saber cuántas líneas contienen `ttl=115` dentro del fichero `ping.log`:

```bash
grep -c "ttl=64" ping.log
```

<figure><img src="https://36118879-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F5YP0HIeDL3z0KLMojf1y%2Fuploads%2F23KQ7HwS89dzTZd4YSUd%2Fimage.png?alt=media&#x26;token=68e6e5b5-638a-4c18-80a0-ae5bf4b3d80d" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
**Eliminar un patrón con grep (-v)**

Con el parametro -v grep elimina una palabra que indiquemos del output.

```bash
grep -v body archivo.txt
```
{% endstep %}
{% endstepper %}
