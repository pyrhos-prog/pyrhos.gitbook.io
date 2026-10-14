# Funciones

> Bloques de código con nombre que se pueden invocar varias veces. Evitan repetir código y organizan el script.

### Creación y llamada

#### Sintaxis

```bash
#!/bin/bash

function saludar(){
  echo "Hola, $1"
}

saludar "mundo"
```

Formas equivalentes de declararla:

```bash
function saludar { echo "Hola"; }     # con la palabra function
saludar() { echo "Hola"; }            # estilo POSIX (más portable)
```

#### Reglas

* La función debe estar **definida antes** de llamarla. Por eso las funciones se colocan al principio del script.
* Se invoca escribiendo su nombre como si fuera un comando, sin paréntesis.
* Una función puede llamar a otras funciones.
* Un nombre de función tiene prioridad sobre un comando externo del mismo nombre.

#### Estructura recomendada

```bash
#!/bin/bash

# 1. Funciones
function panel_ayuda(){
  echo "Uso: $0 <fichero>"
}

function procesar(){
  echo "Procesando $1"
}

# 2. Flujo principal
if [[ $# -ne 1 ]]; then
  panel_ayuda
  exit 1
fi

procesar "$1"
```

#### Inspeccionar y eliminar

```bash
declare -f saludar      # muestra el código de la función
declare -F              # lista los nombres de todas las funciones
unset -f saludar        # elimina la función
```

### Argumentos y valores de retorno

#### Argumentos

Dentro de una función, `$1`, `$2`... son los argumentos de la **función**, no los del script.

```bash
function sumar(){
  echo "Recibidos: $#"
  echo "Todos: $@"
  echo "Suma: $(( $1 + $2 ))"
}

sumar 3 4
```

| Variable      | Significado dentro de la función  |
| ------------- | --------------------------------- |
| `$1`, `$2`... | Argumentos de la función          |
| `$#`          | Número de argumentos              |
| `$@`          | Todos los argumentos              |
| `$0`          | Sigue siendo el nombre del script |

Si se necesita usar los argumentos del script dentro de la función, hay que pasarlos: `mi_funcion "$@"`.

#### Valor de retorno: return

`return` devuelve únicamente un **código de salida** (0-255). No sirve para devolver texto ni números grandes.

```bash
function es_par(){
  (( $1 % 2 == 0 ))      # el código de la última orden es el de la función
}

if es_par 4; then
  echo "Es par"
fi

es_par 3
echo $?                  # 1
```

```bash
function existe(){
  [[ -f "$1" ]] || return 1
  return 0
}
```

#### Devolver datos: echo y sustitución de comandos

```bash
function sumar(){
  echo $(( $1 + $2 ))
}

total=$(sumar 3 4)
echo "$total"             # 7
```

Todo lo que la función imprime por salida estándar se captura. Los mensajes de depuración o error deben ir a stderr (`>&2`) para no mezclarse con el resultado.

```bash
function dividir(){
  if [[ $2 -eq 0 ]]; then
    echo "Error: división entre 0" >&2
    return 1
  fi
  echo $(( $1 / $2 ))
}

resultado=$(dividir 10 2) || echo "Falló"
```

#### Resumen

| Necesidad                         | Mecanismo                                  |
| --------------------------------- | ------------------------------------------ |
| Indicar éxito o fallo             | `return 0` / `return 1`                    |
| Devolver un valor                 | `echo` + `$(...)`                          |
| Modificar una variable del script | Variable global (o `declare -n`, avanzado) |

### Variables locales y ámbito

Por defecto, las variables de Bash son **globales**, incluso las creadas dentro de una función.

```bash
function f(){
  x=10
}
f
echo "$x"       # 10 (visible fuera)
```

#### local

`local` limita la variable a la función y a las que esta llame.

```bash
function f(){
  local x=10
  echo "Dentro: $x"
}

x=1
f
echo "Fuera: $x"     # 1 (no se modificó)
```

Reglas:

* `local` solo se puede usar dentro de una función.
* Se recomienda declarar `local` todas las variables internas para evitar efectos colaterales.
* Para evitar perder el código de salida de un comando, declarar y asignar en líneas separadas:

```bash
local resultado
resultado=$(comando_que_puede_fallar)     # $? conserva el código del comando
# local resultado=$(comando)               # $? pasaría a ser el de "local"
```

#### Recursividad

```bash
function factorial(){
  local n=$1
  if (( n <= 1 )); then
    echo 1
  else
    local anterior
    anterior=$(factorial $((n - 1)))
    echo $(( n * anterior ))
  fi
}

factorial 5     # 120>
```

### Reutilización con source

Las funciones de uso común se guardan en un fichero aparte y se cargan con `source` (o `.`).

`utilidades.sh`:

```bash
#!/bin/bash

function log(){
  echo "[$(date +%T)] $*"
}

function error(){
  echo "[ERROR] $*" >&2
  exit 1
}
```

Script que lo usa:

```bash
#!/bin/bash

source ./utilidades.sh

log "Iniciando"
[[ -f datos.txt ]] || error "Falta datos.txt"
```

#### Ruta relativa al propio script

Para que funcione desde cualquier directorio:

```bash
directorio_script="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "$directorio_script/utilidades.sh"
```

#### Comprobar si el fichero existe

```bash
[[ -f ./utilidades.sh ]] || { echo "Falta utilidades.sh" >&2; exit 1; }
source ./utilidades.sh
```

#### Buenas prácticas

* Un fichero de librería solo define funciones y constantes; no ejecuta acciones al cargarse.
* Se le puede dar extensión `.sh` o `.lib` y no necesita permiso de ejecución.
* Evitar nombres de función genéricos que choquen con comandos del sistema (`test`, `log` en algunos entornos): usar un prefijo, por ejemplo `util_log`.
