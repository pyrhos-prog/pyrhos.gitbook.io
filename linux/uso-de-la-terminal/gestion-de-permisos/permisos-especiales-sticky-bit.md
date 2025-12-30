# Permisos especiales - Sticky Bit

{% embed url="https://keepcoding.io/blog/que-es-el-sticky-bit-y-como-configurarlo/" %}

## Sticky Bit

El permiso Sticky bit hace que solo el usuario que ha creado el directorio o archivo pueda eliminar o renombrarlo.

## Ejemplo de uso del permiso Sticky Bit

{% stepper %}
{% step %}
### Primero vamos a crear un directorio con un usuario de prueba:

<pre><code><strong>testuser@debian-laptop:~$ ls -l
</strong>total 4
drwxrwxr-x 2 testuser testuser 4096 dic  7 23:54 test
</code></pre>


{% endstep %}

{% step %}
### Cambiamos los permisos a 777 en el directorio:

```
testuser@debian-laptop:~$ ls -l
total 4
drwxrwxrwx 2 testuser testuser 4096 dic  7 23:55 test
```




{% endstep %}

{% step %}
### Dentro del directorio creamos un archivo:&#x20;

```
testuser@debian-laptop:~/test$ echo "probando permisos" > file.txt 
testuser@debian-laptop:~/test$ ls -l
total 4
-rw-rw-r-- 1 testuser testuser 18 dic  7 23:55 file.txt 
```


{% endstep %}

{% step %}
### Cambiamos a otro usuario y tratamos de borrar el archivo:

<pre><code><strong>pyrhos@debian-laptop:/home/testuser/test$ ls -l
</strong>total 4
-rw-rw-r-- 1 testuser testuser 18 dic  7 23:55 file.txt
pyrhos@debian-laptop:/home/testuser/test$ rm file.txt 
rm: ¿borrar el regular file 'file.txt'  protegido contra escritura? (s/n) s
pyrhos@debian-laptop:/home/testuser/test$ ls -l
total 0
</code></pre>
{% endstep %}
{% endstepper %}

Aunque el archivo no tenga los permisos necesarios para que otro usuario lo pueda borrar, el directorio si, y se sobreponen los archivos del directorio sobre los del archivo, para evitar eso podemos usar el permiso especial Sticky Bit:

```
testuser@debian-laptop:~$ chmod +t test/
testuser@debian-laptop:~$ ls -l
total 4
drwxrwxrwt 2 testuser testuser 4096 dic  7 23:59 test
```

Ahora al intentar borrar el archivo con otro usuario no nos va a dejar aunque el directorio tenga los permisos necesarios, porque tiene el permiso especial:

```
pyrhos@debian-laptop:/home/testuser/test$ ls -l
total 4
-rw-rw-r-- 1 testuser testuser 17 dic  8 00:03 file.txt
pyrhos@debian-laptop:/home/testuser/test$ rm file.txt 
rm: ¿borrar el regular file 'file.txt'  protegido contra escritura? (s/n) s
rm: no se puede borrar 'file.txt': Operación no permitida
```

{% embed url="https://keepcoding.io/blog/que-es-el-sticky-bit-y-como-configurarlo/" %}
