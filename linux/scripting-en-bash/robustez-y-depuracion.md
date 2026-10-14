# Robustez y depuración

> Cómo escribir scripts que fallen de forma controlada, localizar errores y mantener un estilo limpio.

### Modo estricto

Por defecto, Bash continúa tras un error y trata variables sin definir como vacías, lo que puede provocar daños silenciosos.

```bash
#!/bin/bash
set -euo pipefail
```

| Opción            | Efecto                                                          |
| ----------------- | --------------------------------------------------------------- |
| `set -e`          | Aborta el script si un comando devuelve un código distinto de 0 |
| `set -u`          | Error al usar una variable no definida                          |
| `set -o pipefail` | Una tubería falla si falla cualquiera de sus comandos           |

#### Por qué importa

```bash
rm -rf "$directorio/"          # si $directorio está vacío, equivale a rm -rf /
```

Con `set -u`, el script aborta antes de ejecutar esa línea.

#### Matices de set -e

`set -e` **no** aborta en estos casos:

* Comandos dentro de una condición (`if`, `while`, `until`).
* Comandos seguidos de `&&` o `||` (salvo el último).
* Comandos precedidos de `!`.

```bash
set -e
grep "x" fichero || true           # permite que grep no encuentre nada sin abortar
if ! comando; then echo "falló"; fi
```

Otras particularidades:

* `((contador++))` devuelve 1 cuando el valor previo es 0; con `set -e` aborta. Es preferible `contador=$((contador + 1))`.
* Las funciones llamadas dentro de un `if` o con `||` se ejecutan sin los efectos de `set -e`.

#### Variables opcionales con set -u

```bash
nombre="${1:-}"                    # cadena vacía si no se pasó argumento
[[ -n ${DEBUG:-} ]] && echo "debug activo"
```

#### Activar y desactivar por tramos

```bash
set +e                             # desactiva
comando_que_puede_fallar
codigo=$?
set -e                             # reactiva
```

### Depuración

#### Modo traza: set -x

Muestra cada comando tal como se ejecuta, con las variables ya expandidas.

```bash
bash -x script.sh                  # todo el script
```

```bash
#!/bin/bash
set -x                             # activar en un punto
comando_sospechoso
set +x                             # desactivar
```

Mejorar la traza con `PS4`:

```bash
export PS4='+ ${BASH_SOURCE}:${LINENO}: ${FUNCNAME[0]:-main}: '
bash -x script.sh
```

#### Comprobar la sintaxis sin ejecutar

```bash
bash -n script.sh                  # solo analiza; no ejecuta nada
```

#### Técnicas manuales

```bash
echo "DEBUG: variable=$variable" >&2

[[ ${DEBUG:-0} -eq 1 ]] && echo "valor=$valor" >&2
DEBUG=1 ./script.sh                # activar solo cuando se necesita

function debug(){ [[ ${DEBUG:-0} -eq 1 ]] && echo "[DEBUG] $*" >&2; }
```

#### Información útil

| Variable           | Contenido                                  |
| ------------------ | ------------------------------------------ |
| `$LINENO`          | Número de línea actual                     |
| `$FUNCNAME`        | Array con la pila de funciones             |
| `$BASH_SOURCE`     | Nombre del fichero del script              |
| `$?`               | Código del último comando                  |
| `${PIPESTATUS[@]}` | Códigos de cada parte de la última tubería |

#### Aislar el problema

* Reducir el script al mínimo que reproduce el fallo.
* Ejecutar los comandos sospechosos uno a uno en la terminal.
* Comprobar con `echo` qué valor llega realmente a cada variable.
* Revisar las comillas: gran parte de los errores vienen de variables sin entrecomillar.

### Gestión de errores y logging

#### Comprobar el resultado de los comandos

```bash
if ! cp origen destino; then
  echo "No se pudo copiar" >&2
  exit 1
fi

cd "$directorio" || { echo "No existe $directorio" >&2; exit 1; }
```

#### Función de error

```bash
function error(){
  echo "[ERROR] $*" >&2
  exit 1
}

[[ -f config.conf ]] || error "Falta config.conf"
```

#### trap ERR

Informa de dónde falló un comando:

```bash
set -eE
trap 'echo "[ERROR] Línea $LINENO: comando fallido (código $?)" >&2' ERR
```

#### Logging

Función con marca de tiempo y nivel:

```bash
log_fichero="./script.log"

function log(){
  local nivel=$1; shift
  printf '%s [%s] %s\n' "$(date '+%F %T')" "$nivel" "$*" | tee -a "$log_fichero" >&2
}

log INFO  "Iniciando proceso"
log WARN  "Disco al 90%"
log ERROR "No se pudo conectar"
```

Redirigir todo el script a un log y a la pantalla:

```bash
exec > >(tee -a script.log) 2>&1
```

Con systemd, los mensajes enviados a stdout/stderr van al journal; con `logger` se envían al registro del sistema:

```bash
logger -t mi_script "Proceso completado"
```

#### Códigos de salida coherentes

```bash
readonly E_USO=2
readonly E_FICHERO=3
readonly E_RED=4

[[ $# -eq 1 ]] || { echo "Uso: $0 <fichero>" >&2; exit $E_USO; }
[[ -f $1 ]]    || { echo "No existe $1" >&2; exit $E_FICHERO; }
```

#### Reintentos

```bash
function reintentar(){
  local intentos=$1; shift
  local n
  for ((n = 1; n <= intentos; n++)); do
    "$@" && return 0
    echo "Intento $n/$intentos fallido" >&2
    sleep $((n * 2))
  done
  return 1
}

reintentar 3 curl -sf https://example.com/api
```

### Ficheros temporales con mktemp

Crear ficheros temporales con nombres fijos (`/tmp/datos.txt`) es inseguro: otro usuario puede anticiparse al nombre y provocar sobrescrituras o enlaces simbólicos maliciosos. `mktemp` crea un nombre único y con permisos restringidos.

```bash
tmp=$(mktemp)                          # fichero: /tmp/tmp.XXXXXXXXXX
dir=$(mktemp -d)                       # directorio temporal
tmp=$(mktemp /tmp/mi_script.XXXXXX)    # con plantilla (las X se sustituyen)
```

#### Limpieza garantizada

```bash
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT

echo "datos" > "$tmpdir/entrada.txt"
# ... el directorio se borra siempre al terminar, incluso con error o Ctrl+C
```

Para que `EXIT` cubra también `Ctrl+C`, se convierte la señal en un `exit`:

```bash
trap 'rm -rf "$tmpdir"' EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
```

#### Alternativa: sin ficheros

Process substitution y here-strings evitan crear temporales en muchos casos:

```bash
diff <(sort a.txt) <(sort b.txt)
```

### Análisis estático con ShellCheck

ShellCheck analiza el código sin ejecutarlo y avisa de errores de sintaxis, comillas olvidadas y malas prácticas.

```bash
sudo apt install shellcheck
shellcheck script.sh
shellcheck -s bash script.sh           # forzar dialecto
shellcheck -x script.sh                # seguir los ficheros cargados con source
```

Ejemplo de aviso:

```
In script.sh line 5:
rm $fichero
   ^------^ SC2086: Double quote to prevent globbing and word splitting.
```

#### Avisos frecuentes

| Código   | Problema                                                        |
| -------- | --------------------------------------------------------------- |
| `SC2086` | Variable sin comillas                                           |
| `SC2046` | Sustitución de comandos sin comillas                            |
| `SC2164` | `cd` sin comprobar fallo (usar `cd dir \|\| exit`)              |
| `SC2155` | Declarar y asignar en la misma línea oculta el código de salida |
| `SC2162` | `read` sin `-r`                                                 |
| `SC2034` | Variable definida y no usada                                    |
| `SC2006` | Usar `$(...)` en lugar de comillas invertidas                   |
| `SC2012` | Parsear la salida de `ls` (usar globbing o `find`)              |

#### Ignorar un aviso concreto

```bash
# shellcheck disable=SC2034
variable_usada_en_otro_fichero="valor"
```

Se debe ignorar un aviso solo cuando se ha comprobado que es un falso positivo, y dejando un comentario que lo justifique.

Existe también la versión web (shellcheck.net) y extensiones para los editores más comunes.

### Buenas prácticas y estilo

#### Estructura del script

1. Shebang
2. Comentario de cabecera (qué hace, uso)
3. Modo estricto (`set -euo pipefail`)
4. Constantes y variables globales
5. Funciones
6. `trap`
7. Parseo de argumentos
8. Flujo principal (idealmente en una función `main`)

```bash
#!/bin/bash
# resumen.sh - Descripción breve
set -euo pipefail

readonly VERSION="1.0"

function main(){
  # lógica principal
  echo "Hola"
}

main "$@"
```

#### Reglas prácticas

* Entrecomillar todas las variables: `"$var"`, `"${array[@]}"`.
* Usar `[[ ]]` en lugar de `[ ]`.
* Usar `$(...)` en lugar de comillas invertidas.
* Declarar `local` las variables dentro de funciones.
* Comprobar que existen los ficheros, directorios y dependencias antes de usarlos.
* Usar rutas absolutas o relativas al propio script, no suponer el directorio actual.
* Enviar los mensajes de error a stderr.
* No parsear la salida de `ls`; usar globbing o `find`.
* No usar `cat fichero | comando` cuando `comando < fichero` basta.
* Nombres descriptivos: `ruta_destino` mejor que `rd`.
* Constantes en mayúsculas y `readonly`.
* No dejar credenciales ni datos personales en el código; leerlos del entorno o de ficheros con permisos restringidos.
* Probar con ShellCheck antes de dar por terminado el script.
* Si el script crece mucho o necesita estructuras de datos complejas, valorar otro lenguaje (Python).

#### Seguridad al ejecutar scripts

* No ejecutar scripts descargados sin leerlos (`curl ... | bash` es un riesgo).
* Cuidado con `eval`: ejecuta cualquier cadena, incluida la que provenga de una entrada externa.
* Validar siempre los datos externos (argumentos, ficheros, respuestas de red) antes de usarlos en comandos.
* Establecer un `PATH` explícito en scripts que se ejecutan con privilegios.
* Permisos mínimos: `chmod 700` para scripts privados.

### Errores comunes

#### Sintaxis

| Error                         | Corrección                                      |
| ----------------------------- | ----------------------------------------------- |
| `nombre = "x"`                | Sin espacios: `nombre="x"`                      |
| `[[$a -eq 1]]`                | Con espacios: `[[ $a -eq 1 ]]`                  |
| `if [[ ... ]] then`           | Falta `;` antes de `then`: `if [[ ... ]]; then` |
| Olvidar `fi`, `done` o `esac` | Cada estructura se cierra                       |
| `{ comando }`                 | Requiere `;` antes de `}`: `{ comando; }`       |
| Olvidar `:` en getopts        | `m:` cuando la opción necesita argumento        |

#### Comillas y expansión

| Error                          | Corrección                       |
| ------------------------------ | -------------------------------- |
| `rm $fichero`                  | `rm "$fichero"`                  |
| `for f in $(ls)`               | `for f in *`                     |
| `for x in ${array[@]}`         | `for x in "${array[@]}"`         |
| `[ $a = b ]` con `$a` vacío    | `[[ $a == b ]]` o `[ "$a" = b ]` |
| `echo $var` con espacios o `*` | `echo "$var"`                    |

#### Lógica

| Error                                          | Corrección                                        |
| ---------------------------------------------- | ------------------------------------------------- |
| Comparar números con `==` o cadenas con `-eq`  | `-eq` para números, `==` para cadenas (o `(( ))`) |
| `cd dir` sin comprobar                         | `cd dir \|\| exit 1`                              |
| `cmd1 && cmd2 \|\| cmd3` como si fuera `if`    | Usar un `if` real                                 |
| Variables modificadas en `cat f \| while read` | `while read ... done < f`                         |
| Función usada antes de definirla               | Definirla antes de la llamada                     |
| `trap` sin `exit` en la función                | Añadir `exit` o el script continúa                |
| Cursor oculto con `tput civis` y no restaurado | `tput cnorm` en la limpieza                       |
| `((i++))` con `set -e` cuando `i` es 0         | `i=$((i + 1))`                                    |
| `local var=$(cmd)` oculta el código de `cmd`   | Declarar y asignar por separado                   |

#### Entorno

| Error                                                    | Causa y solución                                             |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `Permission denied`                                      | Falta `chmod +x`                                             |
| `command not found` al ejecutar                          | Usar `./script.sh`                                           |
| `bad interpreter`                                        | Shebang incorrecto o finales de línea CRLF (`dos2unix`)      |
| Funciona a mano pero no en cron                          | Entorno mínimo: rutas absolutas y `PATH` explícito           |
| Funciona con `bash script.sh` pero no con `sh script.sh` | El script usa características de Bash; `sh` puede ser `dash` |
| Resultados distintos entre sistemas                      | Diferencias entre GNU y BSD (`sed -i`, `date`, `getopt`)     |
