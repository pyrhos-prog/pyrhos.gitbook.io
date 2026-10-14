# Shell y sistema de ficheros

Uso diario de la shell: moverse por el sistema, gestionar ficheros y permisos, comodines, entorno y configuración.

### Navegación y gestión de ficheros

#### Estructura de directorios

| Ruta            | Contenido                                                            |
| --------------- | -------------------------------------------------------------------- |
| `/`             | Raíz del sistema.                                                    |
| `/home`         | Directorios personales de los usuarios.                              |
| `/root`         | Directorio personal del usuario root.                                |
| `/etc`          | Ficheros de configuración.                                           |
| `/var`          | Datos variables: logs, colas, cachés.                                |
| `/tmp`          | Temporales (se limpian al reiniciar o periódicamente).               |
| `/usr`          | Programas y librerías.                                               |
| `/bin`, `/sbin` | Binarios esenciales (en muchas distribuciones enlaces a `/usr/bin`). |
| `/opt`          | Software adicional.                                                  |
| `/dev`          | Ficheros de dispositivos (`/dev/null`, `/dev/tcp`).                  |
| `/proc`         | Información del kernel y procesos en forma de ficheros.              |

#### Rutas

* Absoluta: empieza por `/` (`/etc/hosts`).
* Relativa: depende del directorio actual (`../datos/fichero.txt`).
* `.` directorio actual, `..` directorio padre, `~` directorio personal, `-` directorio anterior (`cd -`).

#### Comandos esenciales

```bash
pwd                       # directorio actual
cd /etc                   # cambiar de directorio
cd                        # ir al directorio personal
cd -                      # volver al directorio anterior

ls                        # listar
ls -la                    # largo, incluye ocultos
ls -lh                    # tamaños legibles
ls -lt                    # ordenado por fecha

mkdir proyecto            # crear directorio
mkdir -p a/b/c            # crear con padres

touch fichero.txt         # crear vacío o actualizar fecha
cp origen destino         # copiar
cp -r carpeta copia       # copiar directorio
mv antiguo nuevo          # mover o renombrar
rm fichero                # borrar fichero
rm -r carpeta             # borrar directorio y contenido
rmdir carpeta_vacia       # borrar directorio vacío

cat fichero               # mostrar contenido
file fichero              # tipo de fichero
stat fichero              # metadatos detallados
ln -s destino enlace      # enlace simbólico
```

`rm` no tiene papelera. Conviene usar `rm -i` o comprobar antes con `ls` el patrón que se va a borrar.

Los ficheros y directorios cuyo nombre empieza por `.` están ocultos y solo aparecen con `ls -a`.

### Permisos y propietarios

Salida de `ls -l`:

```
-rwxr-xr--  1  usuario  grupo  1024  ene 1 10:00  script.sh
│└┬┘└┬┘└┬┘
│ │  │  └── permisos de otros
│ │  └───── permisos del grupo
│ └──────── permisos del propietario
└────────── tipo (- fichero, d directorio, l enlace)
```

#### Significado

| Permiso | En un fichero  | En un directorio                          |
| ------- | -------------- | ----------------------------------------- |
| `r` (4) | Leer contenido | Listar contenido                          |
| `w` (2) | Modificar      | Crear, borrar y renombrar dentro          |
| `x` (1) | Ejecutar       | Entrar (`cd`) y acceder a lo que contiene |

#### chmod

Modo octal (suma de 4, 2 y 1 para propietario, grupo y otros):

```bash
chmod 755 script.sh     # rwxr-xr-x
chmod 644 fichero.txt   # rw-r--r--
chmod 600 clave.txt     # rw-------
```

Modo simbólico:

```bash
chmod +x script.sh      # añadir ejecución a todos
chmod u+x script.sh     # solo al propietario
chmod g-w fichero       # quitar escritura al grupo
chmod o= fichero        # quitar todo a otros
chmod -R 750 carpeta    # recursivo
```

#### Propietarios

```bash
chown usuario fichero
chown usuario:grupo fichero
chown -R usuario:grupo carpeta
chgrp grupo fichero

id                      # usuario, uid, grupos
whoami
```

#### umask

Define los permisos que se quitan al crear ficheros nuevos.

```bash
umask          # 0022 es habitual
# ficheros: 666 - 022 = 644      directorios: 777 - 022 = 755
```

#### Permisos especiales

| Bit    | Efecto                                                                                   | Octal |
| ------ | ---------------------------------------------------------------------------------------- | ----- |
| SUID   | El fichero se ejecuta con los privilegios de su propietario                              | 4000  |
| SGID   | Ejecución con el grupo del fichero; en directorios, los nuevos ficheros heredan el grupo | 2000  |
| Sticky | En un directorio, solo el propietario puede borrar sus ficheros (`/tmp`)                 | 1000  |

### Globbing y comodines

El globbing lo expande la shell **antes** de ejecutar el comando.

| Patrón              | Coincide con                                  |
| ------------------- | --------------------------------------------- |
| `*`                 | Cualquier cadena (incluida vacía)             |
| `?`                 | Un único carácter                             |
| `[abc]`             | Uno de los caracteres indicados               |
| `[a-z]`             | Un carácter del rango                         |
| `[!abc]` o `[^abc]` | Cualquier carácter excepto los indicados      |
| `{a,b,c}`           | Expansión de llaves (genera cada alternativa) |

```bash
ls *.txt                 # todos los .txt
ls fichero?.txt          # fichero1.txt, ficheroa.txt
ls [0-9]*                # empiezan por un dígito
ls *.{jpg,png}           # ambos formatos
cp informe.txt{,.bak}    # equivale a: cp informe.txt informe.txt.bak
```

Notas:

* `*` no incluye ficheros ocultos. Para incluirlos: `shopt -s dotglob`.
* Si el patrón no coincide con nada, Bash lo deja literal (salvo con `shopt -s nullglob`, que lo vacía).
* Para evitar la expansión, se entrecomilla: `find . -name "*.txt"`.
* Globbing y expresiones regulares son cosas distintas: `*` en glob significa "cualquier cosa"; en regex significa "repetir lo anterior".

### Variables de entorno y PATH

Las variables de entorno se heredan por los procesos hijos.

```bash
env                      # listar todas
printenv HOME            # una concreta
echo "$HOME" "$USER" "$PWD" "$SHELL"

export MI_VAR="valor"    # exportar a procesos hijos
MI_VAR=valor ./script.sh # solo para esa ejecución
unset MI_VAR
```

#### PATH

Lista de directorios (separados por `:`) donde la shell busca ejecutables.

```bash
echo "$PATH"
# /usr/local/bin:/usr/bin:/bin

export PATH="$PATH:$HOME/bin"      # añadir un directorio al final
export PATH="$HOME/bin:$PATH"      # añadir al principio (prioridad)

command -v python3                 # qué ejecutable se usaría
```

Para que el cambio sea permanente se añade la línea a `~/.bashrc`.

Variables comunes:

| Variable         | Contenido                    |
| ---------------- | ---------------------------- |
| `HOME`           | Directorio personal          |
| `USER`           | Usuario actual               |
| `PWD` / `OLDPWD` | Directorio actual / anterior |
| `SHELL`          | Shell por defecto            |
| `PATH`           | Rutas de búsqueda            |
| `EDITOR`         | Editor por defecto           |
| `LANG`           | Idioma y codificación        |

### Alias, historial y atajos

#### Alias

```bash
alias ll='ls -lah'
alias actualizar='sudo apt update && sudo apt upgrade'
alias                 # listar los definidos
unalias ll            # eliminar
```

Para que persistan se guardan en `~/.bashrc`. Los alias no se expanden dentro de scripts no interactivos; ahí se usan funciones.

#### Historial

```bash
history               # lista numerada
history | grep ssh    # buscar un comando previo
!!                    # repetir el último comando
!42                   # repetir el comando número 42
!ssh                  # repetir el último que empezó por "ssh"
!$                    # último argumento del comando anterior
sudo !!               # repetir el anterior con sudo
```

El historial se guarda en `~/.bash_history`. Variables útiles en `~/.bashrc`:

```bash
HISTSIZE=10000
HISTFILESIZE=20000
HISTCONTROL=ignoredups:ignorespace   # ignora duplicados y líneas que empiezan con espacio
```

#### Atajos de teclado

| Atajo               | Acción                                             |
| ------------------- | -------------------------------------------------- |
| `Ctrl+R`            | Búsqueda inversa en el historial                   |
| `Ctrl+A` / `Ctrl+E` | Inicio / final de línea                            |
| `Ctrl+U`            | Borrar desde el cursor hasta el inicio             |
| `Ctrl+K`            | Borrar desde el cursor hasta el final              |
| `Ctrl+W`            | Borrar la palabra anterior                         |
| `Ctrl+L`            | Limpiar pantalla                                   |
| `Ctrl+C`            | Interrumpir el comando en ejecución                |
| `Ctrl+Z`            | Suspender el comando (pasa a segundo plano parado) |
| `Ctrl+D`            | Fin de entrada / cerrar la shell                   |
| `Tab`               | Autocompletar (doble `Tab` muestra opciones)       |
| `Alt+.`             | Insertar el último argumento del comando anterior  |

### Ficheros de configuración

Bash lee distintos ficheros según cómo se inicie.

| Tipo de shell        | Cuándo ocurre                                        | Ficheros que lee                                                                                |
| -------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Login                | Inicio de sesión (consola, SSH, `bash -l`)           | `/etc/profile`, luego el primero que exista de `~/.bash_profile`, `~/.bash_login`, `~/.profile` |
| No login interactiva | Abrir una terminal en el escritorio, ejecutar `bash` | `~/.bashrc`                                                                                     |
| No interactiva       | Ejecución de un script                               | Ninguno (salvo la variable `BASH_ENV`)                                                          |

Configuración habitual:

* `~/.bashrc`: alias, funciones, prompt, historial, `PATH` para sesiones interactivas.
* `~/.bash_profile` o `~/.profile`: variables de entorno de sesión. Es común que `~/.bash_profile` cargue `~/.bashrc`:

```bash
# ~/.bash_profile
if [[ -f ~/.bashrc ]]; then
  source ~/.bashrc
fi
```

* `~/.bash_logout`: se ejecuta al cerrar una sesión login.

Aplicar cambios sin reiniciar la terminal:

```bash
source ~/.bashrc
```

Personalizar el prompt:

```bash
PS1='\u@\h:\w\$ '    # usuario@equipo:ruta$
```
