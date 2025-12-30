# Comando find

El comando <kbd>find</kbd> sirve para buscar archivos y directorios a través de los parámetros que le indiques, puedes buscar archivos según su tamaño, fecha de creación, nombre, también puedes combinar parámetros.

{% hint style="info" %}
después de poner <kbd>find</kbd> hay que poner la ruta desde la que se va a buscar el archivo.

Ejemplos:

`find /home/usuario/Documentos/ -name "*.pdf"`

`find . -name "archivo.txt"`  el "." representa el directorio actual en el que estas.
{% endhint %}

### Búsqueda por nombre&#x20;

`find  / -name "nombre"`&#x20;

`find  / -name "*.extensión"`  De esta forma puedes buscar todos los archivos con la extension que quieras.

### Búsqueda por tipo

`find  / -type -f -name "nombre"`  el parámetro <kbd>-f</kbd>es para filtrar por archivos

`find  / -type -d -name "nombre"`  el parámetro <kbd>-d</kbd> es para filtrar por directorios

### Búsqueda por tamaño

`find  / -size -50M`  Filtra por archivos de un tamaño menor a 50 megas.

`find  / -size +50M`  Filtra por archivos de un tamaño mayor a 50 megas.



### Búsqueda por fecha

**Filtrar por última modificación**

`find  / -mtime -7` Modificados en los últimos 7 dias.

`find  / -mtime +7` Modificados hace mas de 7 dias.

**Filtrar por último acceso**

`find  / -atime -4`  Ultimo acceso hace menos de 4 dias.

`find  / -atime +4` Ultimo acceso hace menos de 4 dias.

**Filtrar por último cambio de metadatos**

`find  / -ctime -3`  Ultimo cambio de metadatos hace menos de 3 dias.

`find  / -ctime +3`  Ultimo  cambio de metadatos hace menos de 3 dias.



### Búsqueda por permisos

`find / -perm 777`&#x20;

`find  . -perm -111` Busqueda de ejecutables



#### **Ejecutar acciones sobre los archivos encontrados**

**Usar** `-print`, `-delete`, `-exec`

`find . -name "*.tmp" -delete` Eliminar archivos segun su nombre

`find . -name "*.txt" -exec cp {} /backup ;`  Ejecuta un comando



