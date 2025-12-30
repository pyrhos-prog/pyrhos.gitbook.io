# Permisos

Los permisos controlan quien puede leer, escribir y ejecutar un archivo o un directorio.

El primer carácter de los archivos puede ser el siguiente:

| Permisos | Identifica                                                         |
| -------- | ------------------------------------------------------------------ |
| –        | Archivo                                                            |
| d        | Directorio                                                         |
| b        | Archivo de bloques especiales (Archivos especiales de dispositivo) |
| c        | Archivo de caracteres especiales (Dispositivo tty, impresora…)     |
| l        | Archivo de vinculo o enlace (soft/symbolic link)                   |
| p        | Archivo especial de cauce (pipe o tubería)                         |

**El resto de los 9 carácteres esta dividido en 3 secciones.**

<div data-full-width="false" data-with-frame="true"><figure><img src="../../.gitbook/assets/image (12).png" alt="" width="375"><figcaption></figcaption></figure></div>

1. Permisos de usuario
2. Permisos de grupo
3. Permisos de otros usuarios

**Los permisos se representan con :**

r = permiso de lectura

w = permiso de escritura

x = permiso de ejecución
