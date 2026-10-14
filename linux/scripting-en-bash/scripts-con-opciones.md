# Scripts con opciones

## Scripts con opciones

> Cómo construir scripts con interfaz de línea de comandos: menús de opciones con `getopts`, panel de ayuda, validación, colores y convenciones.

### Menú de opciones con getopts

`getopts` procesa las opciones que se pasan al script (`-m`, `-h`...). Se usa dentro de un `while` con un `case`.

#### Sintaxis

```bash
while getopts "m:uh" arg; do
  case $arg in
    m) modo=$OPTARG ;;
    u) usuario=1 ;;
    h) ;;
  esac
done
```

* Cada letra es una opción.
* Una letra seguida de `:` indica que la opción **requiere argumento** (`m:` significa `-m valor`).
* El argumento queda en `$OPTARG`.
* `$arg` contiene la letra de la opción que se está procesando.

#### Variables de getopts

| Variable | Contenido                                              |
| -------- | ------------------------------------------------------ |
| `OPTARG` | Argumento de la opción actual                          |
| `OPTIND` | Índice del siguiente argumento a procesar              |
| `OPTERR` | Si vale 0, se silencian los errores propios de getopts |

#### Modo silencioso

Si la cadena de opciones empieza por `:` (por ejemplo `":m:h"`), getopts no imprime errores y se gestionan manualmente:

* `\?`: opción desconocida (la letra queda en `$OPTARG`).
* `:`: falta el argumento de una opción (la letra queda en `$OPTARG`).

```bash
while getopts ":m:h" arg; do
  case $arg in
    m) modo=$OPTARG ;;
    h) panel_ayuda; exit 0 ;;
    \?) echo "Opción inválida: -$OPTARG" >&2; exit 1 ;;
    :)  echo "La opción -$OPTARG requiere un argumento" >&2; exit 1 ;;
  esac
done
```

#### Patrón: menú que lanza funciones

Cada opción llama a una función existente. Un contador permite detectar si no se pasó ninguna opción.

```bash
#!/bin/bash

function panel_ayuda(){
  echo -e "\n[+] Uso: $0 -m <modo>"
  echo -e "\t-m  modo (listar | escanear)"
  echo -e "\t-h  mostrar este panel\n"
}

function listar(){ echo "Listando..."; }
function escanear(){ echo "Escaneando..."; }

declare -i contador=0

while getopts "m:h" arg; do
  case $arg in
    m) modo=$OPTARG; let contador+=1 ;;
    h) ;;
  esac
done

if [[ $contador -eq 1 ]]; then
  case $modo in
    listar)   listar ;;
    escanear) escanear ;;
    *)        panel_ayuda ;;
  esac
else
  panel_ayuda
fi
```

Uso:

```bash
./script.sh -m listar
./script.sh -h
```

Notas:

* `declare -i` fuerza la variable a entero.
* `let contador+=1` incrementa; equivale a `contador=$((contador+1))`.
* `$0` es el nombre del propio script.
* Las opciones sin argumento se pueden agrupar (`-ab` equivale a `-a -b`).
* Tras el bucle, los argumentos restantes (no opciones) se recuperan con `shift $((OPTIND - 1))`.
* `getopts` solo entiende opciones cortas. Para `--modo` se usa `getopt` (ver sección 6).

### Panel de ayuda y validación de argumentos

#### Panel de ayuda

Debe mostrarse con `-h`, y también cuando el uso es incorrecto.

```bash
function panel_ayuda(){
  cat <<EOF

Uso: $0 [opciones]

Opciones:
  -m <modo>     Modo de ejecución (listar | escanear)
  -f <fichero>  Fichero de entrada
  -v            Salida detallada
  -h            Muestra esta ayuda

Ejemplos:
  $0 -m listar
  $0 -m escanear -f datos.txt

EOF
}
```

#### Validar argumentos

```bash
# Obligatorio presente
if [[ -z $fichero ]]; then
  echo "Falta el fichero (-f)" >&2
  panel_ayuda
  exit 1
fi

# El fichero existe y es legible
if [[ ! -r $fichero ]]; then
  echo "No se puede leer: $fichero" >&2
  exit 1
fi

# Valor numérico
if ! [[ $puerto =~ ^[0-9]+$ ]] || (( puerto < 1 || puerto > 65535 )); then
  echo "Puerto no válido: $puerto" >&2
  exit 1
fi

# Valor dentro de una lista
case $modo in
  listar|escanear) ;;
  *) echo "Modo no válido: $modo" >&2; exit 1 ;;
esac
```

#### Argumentos posicionales sin opciones

```bash
if [[ $# -ne 2 ]]; then
  echo "Uso: $0 <origen> <destino>" >&2
  exit 1
fi
```

#### Valores por defecto

```bash
modo="listar"
verbose=0
while getopts ":m:v" arg; do
  case $arg in
    m) modo=$OPTARG ;;
    v) verbose=1 ;;
  esac
done
```

#### Comprobar dependencias

```bash
for cmd in curl jq; do
  command -v "$cmd" &>/dev/null || { echo "Falta $cmd" >&2; exit 1; }
done
```

### Colores y formato en terminal

Los colores se aplican con secuencias de escape ANSI.

#### Definir variables

```bash
rojo="\e[0;31m"
verde="\e[0;32m"
amarillo="\e[0;33m"
azul="\e[0;34m"
negrita="\e[1m"
fin="\e[0m"

echo -e "${verde}[+] Correcto${fin}"
echo -e "${rojo}[!] Error${fin}"
echo -e "${amarillo}[*] Aviso${fin}"
```

#### Tabla de códigos

| Código | Color    | Código | Color   |
| ------ | -------- | ------ | ------- |
| `30`   | Negro    | `34`   | Azul    |
| `31`   | Rojo     | `35`   | Magenta |
| `32`   | Verde    | `36`   | Cian    |
| `33`   | Amarillo | `37`   | Blanco  |

Formato: `\e[ESTILO;COLORm`, donde estilo `0` normal, `1` negrita, `4` subrayado. `\e[0m` restablece. Sumando 60 al color se obtiene la versión brillante (`91` rojo brillante). Para fondos se suma 10 (`41` fondo rojo).

#### Funciones de mensajes

```bash
function ok(){    echo -e "${verde}[+]${fin} $*"; }
function aviso(){ echo -e "${amarillo}[*]${fin} $*"; }
function error(){ echo -e "${rojo}[!]${fin} $*" >&2; }
```

#### Con printf

```bash
printf "${verde}%s${fin}\n" "Texto en verde"
```

#### Desactivar colores cuando no hay terminal

Si la salida va a un fichero o a otro comando, los códigos ensucian el resultado.

```bash
if [[ ! -t 1 ]]; then
  rojo=""; verde=""; amarillo=""; fin=""
fi
```

### Control del cursor y pantalla con tput

`tput` consulta la base de datos de terminales y emite las secuencias correctas.

| Comando                    | Efecto                                 |
| -------------------------- | -------------------------------------- |
| `tput civis`               | Oculta el cursor                       |
| `tput cnorm`               | Muestra el cursor                      |
| `tput clear`               | Limpia la pantalla                     |
| `tput cup F C`             | Mueve el cursor a la fila F, columna C |
| `tput cols` / `tput lines` | Ancho y alto de la terminal            |
| `tput bold`                | Activa negrita                         |
| `tput setaf N`             | Color de texto (0-7)                   |
| `tput sgr0`                | Restablece atributos                   |

```bash
tput civis
for i in {1..10}; do
  tput cup 5 10
  echo "Progreso: $((i * 10))%"
  sleep 0.3
done
tput cnorm
```

Colores con tput:

```bash
echo "$(tput setaf 2)Verde$(tput sgr0) y $(tput bold)negrita$(tput sgr0)"
```

Si se oculta el cursor, hay que restaurarlo siempre al terminar, también al interrumpir con Ctrl+C (ver tema _Señales y procesos_):

```bash
trap 'tput cnorm' EXIT
```

#### Barra de progreso sencilla

```bash
function barra(){
  local actual=$1 total=$2 ancho=30
  local lleno=$(( actual * ancho / total ))
  printf "\r[%-${ancho}s] %3d%%" "$(printf '#%.0s' $(seq 1 $lleno))" $(( actual * 100 / total ))
}

for i in {1..20}; do barra "$i" 20; sleep 0.1; done; echo
```

### Códigos de salida y convenciones de CLI

#### Códigos de salida

| Código    | Uso recomendado                        |
| --------- | -------------------------------------- |
| `0`       | Éxito                                  |
| `1`       | Error general                          |
| `2`       | Uso incorrecto (argumentos o sintaxis) |
| `64`-`78` | Convención `sysexits.h` (opcional)     |
| `130`     | Interrumpido con Ctrl+C                |

Un script debe terminar con `exit N` explícito y coherente para que otros scripts puedan reaccionar:

```bash
if ./mi_script.sh -m listar; then
  echo "Correcto"
else
  echo "Falló con código $?"
fi
```

#### stdout y stderr

* Resultado útil del programa: stdout.
* Errores, avisos y mensajes de progreso: stderr (`>&2`).

Así se puede redirigir el resultado sin perder los errores: `./script.sh > salida.txt`.

#### Convenciones habituales

| Convención                                 | Ejemplo                             |
| ------------------------------------------ | ----------------------------------- |
| `-h` y `--help` muestran la ayuda          | `script.sh -h`                      |
| `-v` verbose, `-q` silencioso              | `script.sh -v`                      |
| `-o` fichero de salida                     | `script.sh -o salida.txt`           |
| `--` marca el final de las opciones        | `script.sh -- -fichero`             |
| Sin argumentos obligatorios, mostrar ayuda | `script.sh`                         |
| Aceptar entrada por stdin si es razonable  | `cat datos \| script.sh`            |
| Modo no interactivo disponible             | `-y` para confirmar automáticamente |

#### Versión

```bash
VERSION="1.0.0"
# opción -V
echo "$0 v$VERSION"
```

### Opciones largas con getopt

`getopts` (con s) es un builtin y solo admite opciones cortas. `getopt` es un programa externo (versión mejorada de util-linux en Linux) que admite opciones largas.

```bash
#!/bin/bash

function panel_ayuda(){
  echo "Uso: $0 [-m|--modo <modo>] [-v|--verbose] [-h|--ayuda] fichero..."
}

opciones=$(getopt -o m:vh --long modo:,verbose,ayuda -n "$0" -- "$@") || exit 2
eval set -- "$opciones"

verbose=0
while true; do
  case "$1" in
    -m|--modo)    modo=$2; shift 2 ;;
    -v|--verbose) verbose=1; shift ;;
    -h|--ayuda)   panel_ayuda; exit 0 ;;
    --)           shift; break ;;
    *)            echo "Error interno" >&2; exit 2 ;;
  esac
done

# Lo que queda en "$@" son los argumentos posicionales
for fichero in "$@"; do
  echo "Procesando $fichero"
done
```

Uso:

```bash
./script.sh --modo listar -v datos1.txt datos2.txt
./script.sh -m listar --verbose datos.txt
```

Puntos clave:

* `-o` define las opciones cortas, `--long` las largas; los `:` indican argumento obligatorio.
* `getopt` reordena los argumentos y los normaliza; `eval set --` los reasigna a `$@`.
* `shift 2` consume la opción y su valor; `shift`, solo la opción.
* La versión de `getopt` de macOS/BSD no admite opciones largas; en ese caso se procesan a mano con un `while` y `case`:

```bash
while [[ $# -gt 0 ]]; do
  case $1 in
    --modo)    modo=$2; shift 2 ;;
    --modo=*)  modo=${1#*=}; shift ;;
    --verbose) verbose=1; shift ;;
    *)         break ;;
  esac
done
```

#### getopts frente a getopt

|                 | `getopts`       | `getopt`               |
| --------------- | --------------- | ---------------------- |
| Tipo            | Builtin de Bash | Programa externo       |
| Opciones largas | No              | Sí (versión GNU)       |
| Portabilidad    | Alta (POSIX)    | Variable según sistema |
| Complejidad     | Baja            | Media                  |
