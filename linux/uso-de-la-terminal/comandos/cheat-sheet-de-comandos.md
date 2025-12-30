# Cheat sheet de comandos

### Moverse por la terminal

* `pwd` para mostrar la ruta del directorio actual.
* `ls` para listar archivos y directorios en el directorio actual.
* `cd` para cambiar de directorio, por ejemplo, `cd Documentos`.

### Crear y modificar archivos y directorios

* `mkdir` para crear un nuevo directorio, como `mkdir Carpeta`.
* `touch` para crear un nuevo archivo, por ejemplo, `touch nuevo.txt`.
* `rm` para borrar un archivo, como `rm archivo.txt`.
* `rmdir` para eliminar un directorio vacío, por ejemplo, `rmdir Carpeta`.
* `mv` para mover o renombrar archivos o directorios, como `mv archivo.txt nuevo_nombre.txt`.
* `cp` para copiar archivos o directorios, por ejemplo, `cp archivo.txt copia.txt`.
* `cat` para mostrar el contenido de un archivo, como `cat archivo.txt`.
* `nano` o `vim` para editar archivos mediante un editor de texto en la consola.
* `chmod` para cambiar los permisos de un archivo, por ejemplo, `chmod +x script.sh`.
* `./` para ejecutar un script en el directorio actual, como `./script.sh`.
* `find` para buscar archivos por nombre, por ejemplo, `find . -name archivo.txt`.
* `grep` para buscar contenido dentro de archivos, como `grep "palabra" archivo.txt`.

### Procesos

* `ps` para mostrar los procesos en ejecución.
* `top` para obtener una vista en tiempo real del rendimiento del sistema y los procesos.
* `exit` para cerrar la sesión actual.
* `shutdown` o `reboot` para apagar o reiniciar el sistema.
* `adduser` para crear un nuevo usuario, por ejemplo, `adduser nombre_usuario`.
* `deluser` para eliminar un usuario, como `deluser nombre_usuario`.
* `ping` para verificar la conexión a Internet, por ejemplo, `ping www.ejemplo.com`.
* `ssh` para conectarse de forma segura a otro equipo, como `ssh usuario@ip`.



### Gestión del sistema

* `df -h` para ver el espacio libre en los discos duros.
* `free -h` para verificar el uso de memoria RAM.
* `du -sh foldername` para conocer el espacio ocupado por un directorio.
* `history` para ver el historial de comandos.
* `man` para acceder a las páginas de ayuda de los comandos.
* `which` para mostrar la ruta completa al binario de un comando.
* `whereis` para localizar archivos binarios, de ayuda o fuentes.

### Comandos avanzados

* `tar` para crear y extraer archivos comprimidos.
* `gzip` y `gunzip` para comprimir y descomprimir archivos.
* `ssh-keygen` para generar claves SSH.
* `scp` para copiar archivos entre sistemas de forma segura.
* `cron` para programar tareas automatizadas.
* `notify-send` para crear notificaciones en el escritorio.
