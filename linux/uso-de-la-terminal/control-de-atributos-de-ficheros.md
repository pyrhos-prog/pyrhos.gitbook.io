---
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Control de atributos de ficheros

> Los atributos de un fichero representan que tipo de características o propiedades extras posee un archivo o directorio

### &#x20;Gestionar los atributos - chattr

Para gestionar los atributos de un fichero se utiliza `chattr`&#x20;

Podemos añadir el atributo de inmutabilidad para que nadie, ni siquiera el usuario root pueda cambiar o borrar el archivo o directorio

```
pyrhos@debian-laptop:~/Documentos$ chattr +i -V file.txt 
[sudo] contraseña para pyrhos: 
chattr 1.47.2 (1-Jan-2025)
Flags of file.txt set as ----i---------e-------
```

* con +i añadimos al fichero el atributo para que sea inmutable
* -V es para que muestre el verbose al ejecutar el comando

| Atributos | explicación                                                                                                                                                                                                                                                                                                                                           |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A         | El tiempo (tiempo de acceso) del archivo no se puede modificar, lo que puede reducir la cantidad de E / S de disco, lo que es beneficioso para la computadora portátil al mejorar la duración de la batería.                                                                                                                                          |
| S         | Opción de sincronización de E / S del disco duro, función similar a la sincronización                                                                                                                                                                                                                                                                 |
| a         | Es decir, después de configurar este parámetro, solo puede agregar datos al archivo, pero no eliminarlo. Se utiliza principalmente para la seguridad del archivo de registro del servidor. Solo el root puede establecer este atributo.                                                                                                               |
| i         | El archivo no se puede eliminar, renombrar ni vincular, y el contenido no se puede escribir ni agregar (incluso si es un usuario root). Solo root puede establecer esta propiedad                                                                                                                                                                     |
| c         | Comprimir, el archivo se comprimirá automáticamente y luego se almacenará, y se descomprimirá automáticamente cuando se lea                                                                                                                                                                                                                           |
| d         | Eso no es un volcado, el archivo de configuración no puede ser el objetivo de respaldo del programa de volcado                                                                                                                                                                                                                                        |
| j         | Es decir, diario, establezca este parámetro para que cuando el sistema de archivos se monte con el parámetro de montaje "datos = ordenados" o "datos = reescritura", el archivo se grabará primero cuando se escriba (en el diario). Si el parámetro del sistema de archivos se establece en data = journal, el parámetro es automáticamente inválido |
| s         | Esa es una opción segura y confidencial. Cuando se elimina el archivo con el conjunto de atributos s, todos sus bloques de datos se escribirán en 0                                                                                                                                                                                                   |
| u         | Esa es la opción recuperar, recuperar. Al contrario de s, cuando se elimina un archivo, se conservan todos sus bloques de datos y el usuario puede restaurar el archivo en el futuro                                                                                                                                                                  |



### Listar los atributos - lsattr

Para ver los atributos de los archivos o directorios se utiliza el comando `lsattr` , estas son sus opciones

| Opciones | explicación                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------- |
| -R       | Visualice recursivamente los atributos de todos los subdirectorios y archivos en el directorio    |
| -V       | Mostrar información de la versión del programa lsattr                                             |
| -a       | Mostrar información de atributos de todos los archivos, incluidos los archivos que comienzan con. |
| -d       | Mostrar los atributos del directorio, no los atributos de los archivos en el directorio           |
| -v       | Mostrar el número de archivo del documento                                                        |

<br>
