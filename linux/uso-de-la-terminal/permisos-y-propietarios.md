---
icon: terminal
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Permisos y propietarios

### El modelo rwx

Cada fichero/directorio tiene tres niveles de permisos y tres tipos de permiso:

```
-rwxr-xr--
 ___ ___ ___
  |   |   |
  |   |   otros (others)
  |   grupo (group)
  propietario (owner/user)
```

| Permiso       | Fichero                       | Directorio                   |
| ------------- | ----------------------------- | ---------------------------- |
| `r` (read)    | Leer contenido                | Listar contenido (`ls`)      |
| `w` (write)   | Modificar contenido           | Crear/borrar ficheros dentro |
| `x` (execute) | Ejecutar como programa/script | Entrar al directorio (`cd`)  |

&#x20;`x` en un directorio sin `r` permite entrar y acceder a ficheros si conoces el nombre exacto, pero no listarlos con `ls`. Es la base de los directorios "ocultos por permisos" que te puedes encontrar en una escalada.

### Notación octal vs simbólica

Cada permiso vale un número, se suman por nivel:

| Permiso | Valor |
| ------- | ----- |
| `r`     | 4     |
| `w`     | 2     |
| `x`     | 1     |

| Octal | Permisos    | Uso típico                                    |
| ----- | ----------- | --------------------------------------------- |
| `755` | `rwxr-xr-x` | Scripts y ejecutables                         |
| `644` | `rw-r--r--` | Ficheros normales                             |
| `600` | `rw-------` | Ficheros sensibles (claves SSH, credenciales) |
| `700` | `rwx------` | Directorios privados                          |
| `777` | `rwxrwxrwx` | Todo para todos — evitar salvo caso puntual   |

### chmod

```bash
# Modo numérico
chmod 755 script.sh
chmod 644 fichero.txt
chmod 600 id_rsa
chmod -R 755 carpeta/          # recursivo

# Modo simbólico
chmod u+x script.sh             # añade ejecución al propietario (user)
chmod g-w fichero                # quita escritura al grupo
chmod o=r fichero                 # fija "otros" exactamente a solo lectura
chmod a+x script.sh                # añade ejecución a todos (all)
chmod u+x,g+x,o-rwx fichero         # varios cambios separados por comas
```

| Quién              | Letra |
| ------------------ | ----- |
| user (propietario) | `u`   |
| group              | `g`   |
| others             | `o`   |
| all                | `a`   |

| Operador | Efecto                         |
| -------- | ------------------------------ |
| `+`      | Añade el permiso               |
| `-`      | Quita el permiso               |
| `=`      | Fija exactamente esos permisos |

### chown y chgrp

```bash
chown usuario fichero                   # cambia propietario
chown usuario:grupo fichero              # cambia propietario y grupo a la vez
chown -R usuario:grupo carpeta/           # recursivo
chown :grupo fichero                       # cambia solo el grupo (equivalente a chgrp)

chgrp grupo fichero                      # cambia solo el grupo
chgrp -R grupo carpeta/                   # recursivo
```

Normalmente solo root puede cambiar el propietario (`chown usuario`) de un fichero; un usuario normal sí puede cambiar el grupo (`chgrp`) si pertenece al grupo destino.&#x20;

### SUID, SGID y sticky bit

| Permiso especial | Dígito octal extra | Efecto                                                                                                            |
| ---------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| SUID             | 4xxx               | El ejecutable corre con los privilegios del **propietario del fichero**, no del usuario que lo lanza              |
| SGID             | 2xxx               | En ficheros: corre con privilegios del grupo. En directorios: los ficheros nuevos heredan el grupo del directorio |
| Sticky bit       | 1xxx               | En directorios compartidos (`/tmp`), solo el propietario de un fichero puede borrarlo, aunque otros tengan `w`    |

```bash
chmod u+s fichero        # SUID
chmod g+s carpeta/        # SGID
chmod +t carpeta/          # sticky bit
chmod 4755 fichero          # SUID + rwxr-xr-x (equivalente a u+s 755)
```

#### Por qué importa en escalada de privilegios

Si un binario tiene SUID y pertenece a root, ejecutarlo te da capacidades de root mientras corre, aunque tú seas un usuario normal. Es una de las primeras cosas que se buscan en una fase de post-explotación:

```bash
find / -perm -4000 -type f 2>/dev/null       # binarios con SUID
find / -perm -2000 -type f 2>/dev/null       # binarios con SGID
```

Si el binario encontrado está en [GTFOBins](https://gtfobins.github.io/) con función SUID, puede dar shell con privilegios del propietario (normalmente root).

&#x20;`chmod -R 777 carpeta/` es una receta para problemas de seguridad: abre escritura a cualquier usuario del sistema sobre todo el contenido. Evitarlo salvo depuración puntual y revertirlo después.&#x20;

### umask

Máscara que resta permisos por defecto al crear ficheros/directorios nuevos.

```bash
umask           # ver valor actual (ej: 0022)
umask 027         # cambiar (solo para la sesión actual)
```

Con `umask 022`: un fichero nuevo nace con `666 - 022 = 644`, un directorio nuevo con `777 - 022 = 755`.

### Comprobar permisos efectivos

```bash
id                           # UID, GID y grupos del usuario actual
groups usuario                 # grupos a los que pertenece
namei -l /ruta/completa          # desglosa permisos de cada componente de la ruta, útil para depurar "Permission denied"
```
