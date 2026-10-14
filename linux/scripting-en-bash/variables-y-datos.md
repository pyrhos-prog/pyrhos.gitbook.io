# Variables y datos

> Cómo almacenar y manipular información en Bash: variables, comillas, argumentos, entrada del usuario, aritmética, cadenas, arrays y expansiones.

### Variables

Bash no tiene tipos estrictos: todo es texto salvo que se indique lo contrario.

```bash
nombre="mundo"           # asignación: SIN espacios alrededor del =
echo "Hola, $nombre"     # uso con $
echo "${nombre}s"        # llaves para delimitar el nombre
unset nombre             # eliminar
```

Reglas de nombres: letras, números y `_`; no pueden empezar por número; distingue mayúsculas de minúsculas.

#### Atributos con declare

```bash
declare -i numero=5      # entero
declare -r CONSTANTE=10  # solo lectura (equivale a readonly)
declare -x VAR=valor     # exportada (equivale a export)
declare -a lista         # array indexado
declare -A mapa          # array asociativo
```

```bash
readonly MAX=3
MAX=4                    # error: variable de solo lectura
```

#### Resultado de un comando en una variable

```bash
fecha=$(date +%F)
usuarios=$(wc -l < /etc/passwd)
```

#### Errores típicos

```bash
nombre = "x"             # incorrecto: Bash lo interpreta como un comando "nombre"
nombre="x"               # correcto
```

### Comillas y escapes

| Tipo       | Efecto                                                                                    |
| ---------- | ----------------------------------------------------------------------------------------- |
| `'texto'`  | Literal. No expande nada.                                                                 |
| `"texto"`  | Expande variables (`$var`), comandos (`$(...)`) y aritmética; evita separar por espacios. |
| `\c`       | Escapa el carácter siguiente.                                                             |
| `$'texto'` | Interpreta secuencias como `\n`, `\t`.                                                    |

```bash
var="mundo"
echo 'Hola $var'         # Hola $var
echo "Hola $var"         # Hola mundo
echo "Precio: \$5"       # Precio: $5
echo $'Linea1\nLinea2'
```

#### Por qué entrecomillar variables

```bash
fichero="mi documento.txt"
rm $fichero              # intenta borrar "mi" y "documento.txt"
rm "$fichero"            # correcto
```

Regla práctica: entrecomillar siempre `"$variable"` salvo que se quiera explícitamente la separación por palabras o el globbing.

#### echo y printf

```bash
echo -e "a\tb\nc"        # -e interpreta escapes
echo -n "sin salto"      # -n evita el salto de línea
printf '%s tiene %d años\n' "Ana" 30   # más predecible y portable
```

### Argumentos y variables especiales

#### Argumentos posicionales

```bash
#!/bin/bash
echo "Script: $0"
echo "Primero: $1"
echo "Segundo: $2"
echo "Total: $#"
echo "Todos: $@"
```

```bash
./script.sh uno dos
# Script: ./script.sh
# Primero: uno
# Segundo: dos
# Total: 2
```

A partir del décimo argumento hacen falta llaves: `${10}`.

#### Tabla de variables especiales

| Variable               | Contenido                                                         |
| ---------------------- | ----------------------------------------------------------------- |
| `$0`                   | Nombre del script                                                 |
| `$1` ... `$9`, `${10}` | Argumentos posicionales                                           |
| `$#`                   | Número de argumentos                                              |
| `$@`                   | Todos los argumentos (cada uno por separado si va entre comillas) |
| `$*`                   | Todos los argumentos (como una sola cadena si va entre comillas)  |
| `$?`                   | Código de salida del último comando                               |
| `$$`                   | PID del proceso actual                                            |
| `$!`                   | PID del último proceso lanzado en segundo plano                   |

#### "$@" frente a "$\*"

```bash
for arg in "$@"; do echo "[$arg]"; done   # respeta cada argumento, incluso con espacios
```

Casi siempre se usa `"$@"`.

#### shift

Descarta el primer argumento y desplaza los demás.

```bash
while [[ $# -gt 0 ]]; do
  echo "Procesando $1"
  shift
done
```

#### Validar argumentos

```bash
if [[ $# -lt 1 ]]; then
  echo "Uso: $0 <fichero>" >&2
  exit 1
fi
```

### Lectura de datos con read

```bash
read -r nombre                   # lee una línea
read -rp "Nombre: " nombre       # con mensaje
read -rsp "Clave: " clave; echo  # sin eco (el echo final da el salto de línea)
read -t 5 -rp "Responde: " resp  # tiempo límite de 5 s
read -n 1 -rp "¿Seguir? (s/n) " r  # un solo carácter
```

| Opción   | Efecto                                  |
| -------- | --------------------------------------- |
| `-r`     | No interpreta `\` (recomendado siempre) |
| `-p`     | Muestra un mensaje                      |
| `-s`     | Silencioso (no muestra lo escrito)      |
| `-t n`   | Tiempo máximo de espera                 |
| `-n n`   | Lee solo n caracteres                   |
| `-a arr` | Guarda las palabras en un array         |

Sin nombre de variable, el valor queda en `$REPLY`.

#### Separar campos con IFS

```bash
linea="root:x:0:0"
IFS=: read -r usuario _ uid gid <<< "$linea"
echo "$usuario $uid"             # root 0
```

#### Confirmación

```bash
read -rp "¿Continuar? (s/n) " r
[[ $r == [sS] ]] || exit 0
```

### Aritmética

Bash opera solo con enteros.

```bash
a=7; b=2
echo $((a + b))      # 9
echo $((a / b))      # 3  (división entera)
echo $((a % b))      # 1  (resto)
echo $((a ** b))     # 49 (potencia)
```

#### Formas de calcular

```bash
resultado=$((a * b))     # expansión aritmética (la más usada)
(( a++ ))                # evaluación sin devolver valor
(( a > b )) && echo "a es mayor"
let c=a+b                # forma antigua
declare -i d=a+b         # variable entera
```

Incremento: `((contador++))` o `contador=$((contador+1))`.

Con `set -e`, `((x++))` cuando x vale 0 devuelve código 1 y puede abortar el script. Es preferible `x=$((x+1))`.

#### Decimales

Bash no los admite; se delega en otras herramientas:

```bash
echo "scale=2; 7/2" | bc                 # 3.50
awk 'BEGIN { printf "%.2f\n", 7/2 }'     # 3.50
```

#### Números aleatorios y secuencias

```bash
echo $RANDOM                 # 0 a 32767
echo $((RANDOM % 100))       # 0 a 99
seq 1 5                      # 1 2 3 4 5
```

### Cadenas y parameter expansion

Permite manipular el valor de una variable sin comandos externos.

```bash
f="informe.final.txt"
```

#### Longitud y subcadenas

| Expresión  | Resultado   | Significado                                  |
| ---------- | ----------- | -------------------------------------------- |
| `${#f}`    | `17`        | Longitud                                     |
| `${f:0:7}` | `informe`   | Desde posición 0, 7 caracteres               |
| `${f:8}`   | `final.txt` | Desde la posición 8 hasta el final           |
| `${f: -3}` | `txt`       | Últimos 3 caracteres (espacio antes del `-`) |

#### Eliminar patrones

| Expresión  | Resultado       | Significado                             |
| ---------- | --------------- | --------------------------------------- |
| `${f#*.}`  | `final.txt`     | Quita el prefijo más corto que coincida |
| `${f##*.}` | `txt`           | Quita el prefijo más largo (extensión)  |
| `${f%.*}`  | `informe.final` | Quita el sufijo más corto               |
| `${f%%.*}` | `informe`       | Quita el sufijo más largo               |

Regla mnemotécnica: `#` actúa por la izquierda, `%` por la derecha; doble símbolo = coincidencia más larga.

#### Sustituir

```bash
echo "${f/final/inicial}"    # informe.inicial.txt  (primera coincidencia)
echo "${f//./_}"             # informe_final_txt    (todas)
```

#### Mayúsculas y minúsculas (Bash 4+)

```bash
n="Mundo"
echo "${n^^}"    # MUNDO
echo "${n,,}"    # mundo
echo "${n^}"     # primera en mayúscula
```

#### Valores por defecto

| Expresión           | Efecto                                                         |
| ------------------- | -------------------------------------------------------------- |
| `${v:-defecto}`     | Usa `defecto` si `v` está vacía o sin definir (no la modifica) |
| `${v:=defecto}`     | Igual, pero además asigna el valor a `v`                       |
| `${v:?mensaje}`     | Si `v` está vacía, muestra el mensaje y aborta                 |
| `${v:+alternativa}` | Usa `alternativa` solo si `v` tiene valor                      |

```bash
destino="${1:-/tmp}"         # primer argumento o /tmp por defecto
```

#### Comparar cadenas

```bash
[[ $a == $b ]]               # igualdad
[[ $a == pre* ]]             # patrón
[[ $a =~ ^[0-9]+$ ]]         # expresión regular
```

### Arrays

Colección indexada desde 0.

```bash
frutas=("manzana" "pera" "uva")
frutas[3]="kiwi"
frutas+=("melón")            # añadir al final
```

#### Acceso

| Expresión          | Resultado                          |
| ------------------ | ---------------------------------- |
| `${frutas[0]}`     | Primer elemento                    |
| `${frutas[-1]}`    | Último elemento (Bash 4.3+)        |
| `${frutas[@]}`     | Todos los elementos                |
| `${#frutas[@]}`    | Número de elementos                |
| `${!frutas[@]}`    | Índices                            |
| `${frutas[@]:1:2}` | Subconjunto (desde 1, 2 elementos) |

#### Recorrer

```bash
for f in "${frutas[@]}"; do
  echo "$f"
done

for i in "${!frutas[@]}"; do
  echo "$i: ${frutas[$i]}"
done
```

#### Operaciones

```bash
unset 'frutas[1]'                    # eliminar un elemento (deja hueco en los índices)
copia=("${frutas[@]}")               # copiar
mapfile -t lineas < fichero.txt      # cada línea del fichero, un elemento
lineas=($(cat fichero.txt))          # evitar: separa por espacios y aplica globbing
```

Entrecomillar siempre `"${array[@]}"` para conservar elementos con espacios.

### Arrays asociativos

Pares clave-valor. Requieren `declare -A` (Bash 4+).

```bash
declare -A puertos
puertos[ssh]=22
puertos[http]=80
puertos[https]=443

echo "${puertos[ssh]}"               # 22
```

Declaración en una línea:

bash

```bash
declare -A puertos=([ssh]=22 [http]=80 [https]=443)
```

#### Recorrer

bash

```bash
for servicio in "${!puertos[@]}"; do
  echo "$servicio -> ${puertos[$servicio]}"
done
```

#### Operaciones

bash

```bash
echo "${#puertos[@]}"                # número de pares
[[ -v puertos[ftp] ]] || echo "ftp no definido"   # comprobar si existe la clave
unset 'puertos[http]'                # eliminar
```

El orden de recorrido de un array asociativo no está garantizado.

### Expansiones

La shell procesa cada línea aplicando expansiones en este orden aproximado:

1. Llaves `{a,b}`
2. Tilde `~`
3. Parámetros y variables `$var`, aritmética `$(( ))`, sustitución de comandos `$( )`
4. Separación de palabras
5. Globbing `*`, `?`, `[...]`

#### Expansión de llaves

bash

```bash
echo {1..5}              # 1 2 3 4 5
echo {a..e}              # a b c d e
echo {01..10}            # 01 02 ... 10
echo {0..20..5}          # 0 5 10 15 20
echo archivo_{a,b,c}.txt # archivo_a.txt archivo_b.txt archivo_c.txt
mkdir -p proyecto/{src,docs,tests}
```

#### Sustitución de comandos

```bash
echo "Hoy es $(date +%A)"
total=$(ls | wc -l)
```

Se prefiere `$(...)` frente a las comillas invertidas `` `...` ``, porque se puede anidar y es más legible.

#### Tilde

```bash
cd ~          # directorio personal
ls ~/bin
```

#### Expansión aritmética

```bash
echo "Resultado: $((5 * 4))"
```

#### Separación de palabras

El resultado de una expansión sin comillas se separa según `IFS` (espacio, tabulador, salto de línea) y se le aplica globbing. Por eso se entrecomillan las variables.
