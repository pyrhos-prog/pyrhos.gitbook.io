# Descriptores de archivos

Un descriptor de archivo en linux es un número entero que el kernel asigna a cada archivo, dispositivo, socket o flujo abierto por un proceso.

Los descriptores de archivos permiten que los programas lean, escriban o interactúen con archivos y dispositivos sin tener que conocer los detalles del hardware.

***

### **Conceptos clave**

1. **Identificador numérico**
   * Cada archivo abierto recibe un número entero único dentro del proceso.
   * Por ejemplo: `0`, `1`, `2` son los primeros descriptores estándar.
2. **Relación con el kernel**
   * Cuando abres un archivo con `open()`, el kernel devuelve un descriptor.
   * Todas las operaciones de E/S (lectura, escritura, cierre) se realizan usando ese descriptor.
3. **Flujos estándar**\
   Linux define tres descriptores estándar por defecto para cada proceso:

| Descriptor | Nombre | Propósito                         |
| ---------- | ------ | --------------------------------- |
| 0          | stdin  | Entrada estándar (keyboard, pipe) |
| 1          | stdout | Salida estándar (pantalla, pipe)  |
| 2          | stderr | Salida de errores estándar        |

***

### **Operaciones comunes con descriptores de archivos**

1. **Abrir un archivo**

```c
int fd = open("archivo.txt", O_RDONLY);
```

* `fd` es el descriptor que devuelve el kernel.

2. **Leer datos**

```c
char buffer[100];
read(fd, buffer, 100);
```

3. **Escribir datos**

```c
write(fd, "Hola", 4);
```

4. **Cerrar un descriptor**

```c
close(fd);
```

* Libera el descriptor para que pueda usarse en otros archivos.

***

### **Redirección de descriptores en shell**

En la terminal también puedes manipular descriptores:

* Redirigir stdout a un archivo:

```bash
echo "Hola" > salida.txt
```

* Redirigir stderr a un archivo:

```bash
ls /noexiste 2> errores.txt
```

* Redirigir stdout y stderr al mismo archivo:

```bash
comando > todo.txt 2>&1
```

***

### Cheatsheet de descriptores de archivos

{% file src="../.gitbook/assets/bash-redirections-cheat-sheet.pdf" %}
