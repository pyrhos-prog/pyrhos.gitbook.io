# Control de flujo

> Estructuras que deciden qué parte del código se ejecuta y cuántas veces: condicionales, operadores de test, `case`, bucles y menús.

### Condicionales

#### Estructura

```bash
if [[ condición ]]; then
  # código
elif [[ otra condición ]]; then
  # código
else
  # código
fi
```

`if` evalúa el **código de salida** de un comando: 0 se considera verdadero, cualquier otro valor falso. Por eso funciona con cualquier comando, no solo con tests.

```bash
if grep -q "root" /etc/passwd; then
  echo "root existe"
fi

if ! ping -c1 -W1 127.0.0.1 &>/dev/null; then
  echo "Sin respuesta"
fi
```

#### `[[ ]]` frente a `[ ]`

|                                      | `[ ]`                      | `[[ ]]`               |
| ------------------------------------ | -------------------------- | --------------------- |
| Tipo                                 | Comando (`test`), POSIX    | Palabra clave de Bash |
| Variables sin comillas               | Pueden romper la expresión | Seguras               |
| `&&`, `\|\|`                         | No (usa `-a`, `-o`)        | Sí                    |
| Patrones (`== *.txt`) y regex (`=~`) | No                         | Sí                    |

Se recomienda `[[ ]]` en scripts de Bash. Los espacios dentro de los corchetes son obligatorios.

#### Comprobar si se es root

```bash
if [[ $(id -u) -ne 0 ]]; then
  echo "Ejecuta el script como root" >&2
  exit 1
fi
```

### Operadores de test

#### Cadenas

| Operador    | Significado                                     |
| ----------- | ----------------------------------------------- |
| `==` / `!=` | Igual / distinto (acepta patrones con `*`, `?`) |
| `-z "$v"`   | Cadena vacía                                    |
| `-n "$v"`   | Cadena no vacía                                 |
| `<` / `>`   | Orden alfabético (dentro de `[[ ]]`)            |
| `=~`        | Coincide con expresión regular                  |

```bash
[[ $archivo == *.txt ]]          # patrón
[[ $entrada =~ ^[0-9]+$ ]]       # solo dígitos
```

#### Números

| Operador      | Significado           |
| ------------- | --------------------- |
| `-eq`         | Igual                 |
| `-ne`         | Distinto              |
| `-gt` / `-ge` | Mayor / mayor o igual |
| `-lt` / `-le` | Menor / menor o igual |

También se puede usar `(( ))`, más natural para aritmética:

```bash
(( edad >= 18 )) && echo "Mayor de edad"
```

No mezclar: `==` compara cadenas, `-eq` compara números.

#### Ficheros

| Operador                 | Significado                                |
| ------------------------ | ------------------------------------------ |
| `-e f`                   | Existe                                     |
| `-f f`                   | Es fichero regular                         |
| `-d f`                   | Es directorio                              |
| `-L f`                   | Es enlace simbólico                        |
| `-r f` / `-w f` / `-x f` | Permiso de lectura / escritura / ejecución |
| `-s f`                   | Existe y no está vacío                     |
| `f1 -nt f2`              | f1 es más reciente que f2                  |

```bash
if [[ -f "$fichero" && -r "$fichero" ]]; then
  echo "Existe y se puede leer"
fi
```

#### Comprobar variables definidas

```bash
[[ -v variable ]]        # la variable existe (aunque esté vacía)
[[ -z ${variable:-} ]]   # vacía o sin definir
```

### Operadores lógicos

| Operador | Significado                                             |
| -------- | ------------------------------------------------------- |
| `!`      | Negación                                                |
| `&&`     | Y (ejecuta el siguiente solo si el anterior tuvo éxito) |
| `\|\|`   | O (ejecuta el siguiente solo si el anterior falló)      |
| `;`      | Separador (ejecuta siempre el siguiente)                |

#### Dentro de la condición

```bash
if [[ $a -gt 0 && $a -lt 10 ]]; then echo "entre 1 y 9"; fi
if [[ $op == "s" || $op == "S" ]]; then echo "sí"; fi
if [[ ! -d "$dir" ]]; then echo "no es directorio"; fi
```

#### Entre comandos

```bash
mkdir datos && cd datos               # entra solo si se creó
cd "$dir" || exit 1                   # sale si falla el cd
[[ -f config ]] && source config      # carga solo si existe
```

Advertencia con `a && b || c`: `c` se ejecuta también si `b` falla, no solo si `a` falla. Para decisiones reales se usa `if`.

#### Agrupar comandos

```bash
[[ -f "$f" ]] || { echo "No existe $f" >&2; exit 1; }
```

Las llaves requieren un espacio tras `{` y un `;` antes de `}`.

### case

Compara una variable con varios patrones. Es más legible que encadenar `elif`.

```bash
case $opcion in
  start)        echo "Iniciando" ;;
  stop|halt)    echo "Parando" ;;
  [0-9])        echo "Un dígito" ;;
  *.txt)        echo "Fichero de texto" ;;
  *)            echo "Opción no válida" ;;
esac
```

* Cada bloque termina con `;;`.
* `|` separa alternativas.
* Admite patrones de globbing.
* `*)` es el caso por defecto y va al final.

Ejemplo habitual: procesar una respuesta del usuario.

```bash
read -rp "¿Continuar? (s/n) " r
case ${r,,} in
  s|si) echo "Continuando" ;;
  n|no) echo "Cancelado"; exit 0 ;;
  *)    echo "Respuesta no válida"; exit 1 ;;
esac
```

### Bucle for

#### Sobre una lista

```bash
for nombre in ana luis marta; do
  echo "Hola, $nombre"
done
```

#### Sobre un rango

```bash
for i in {1..5}; do echo "$i"; done
for i in {0..20..5}; do echo "$i"; done     # con paso
for i in $(seq 1 5); do echo "$i"; done     # alternativa con seq
```

#### Estilo C

```bash
for ((i = 0; i < 5; i++)); do
  echo "$i"
done
```

#### Sobre ficheros

```bash
for f in *.txt; do
  echo "Procesando $f"
done
```

No se debe iterar sobre `$(ls)`: rompe con nombres con espacios. El globbing directo es la forma correcta.

#### Sobre un array o los argumentos

```bash
for elemento in "${array[@]}"; do echo "$elemento"; done
for arg in "$@"; do echo "$arg"; done
```

### Bucles while y until

`while` repite mientras la condición sea verdadera; `until` repite hasta que lo sea.

```bash
contador=1
while [[ $contador -le 5 ]]; do
  echo "$contador"
  ((contador++))
done

until ping -c1 -W1 127.0.0.1 &>/dev/null; do
  echo "Esperando..."
  sleep 1
done
```

#### Bucle infinito

```bash
while true; do
  # tarea periódica
  sleep 5
done
```

#### Leer un fichero línea a línea

```bash
while IFS= read -r linea; do
  echo "Línea: $linea"
done < fichero.txt
```

* `IFS=` conserva los espacios del principio y final.
* `-r` evita interpretar las barras invertidas.

Atención con `cat fichero | while ...`: el bucle se ejecuta en una subshell y las variables modificadas dentro no se ven fuera. La redirección `done < fichero` evita el problema. Si la entrada es la salida de un comando, se usa `done < <(comando)`.

#### Leer campos separados

```bash
while IFS=: read -r usuario _ uid _; do
  echo "$usuario tiene UID $uid"
done < /etc/passwd
```

### break, continue y códigos de salida

#### break y continue

```bash
for i in {1..10}; do
  [[ $i -eq 3 ]] && continue    # salta a la siguiente iteración
  [[ $i -eq 6 ]] && break       # sale del bucle
  echo "$i"
done
```

`break 2` sale de dos niveles de bucles anidados.

#### Códigos de salida

Todo comando devuelve un número entre 0 y 255. Se consulta con `$?`.

```bash
ls /etc/hosts
echo $?          # 0

ls /no/existe
echo $?          # 2
```

| Código | Significado habitual                  |
| ------ | ------------------------------------- |
| `0`    | Éxito                                 |
| `1`    | Error general                         |
| `2`    | Uso incorrecto del comando            |
| `126`  | Comando encontrado pero no ejecutable |
| `127`  | Comando no encontrado                 |
| `130`  | Terminado con Ctrl+C (128 + señal 2)  |

#### exit y return

```bash
exit 0           # termina el script con éxito
exit 1           # termina con error
return 1         # sale de una función con código 1
```

Un script sin `exit` explícito devuelve el código del último comando ejecutado.

### Menús con select

`select` genera un menú numerado y repite hasta que se hace `break`.

```bash
PS3="Elige una opción: "

select opcion in iniciar parar estado salir; do
  case $opcion in
    iniciar) echo "Iniciando..." ;;
    parar)   echo "Parando..." ;;
    estado)  echo "Todo correcto" ;;
    salir)   break ;;
    *)       echo "Opción no válida" ;;
  esac
done
```

Resultado:

```
1) iniciar
2) parar
3) estado
4) salir
Elige una opción:
```

* `PS3` define el texto del prompt.
* La opción elegida queda en la variable (`$opcion`) y el número escrito en `$REPLY`.
* Se combina con `case` para actuar según la elección.
* Para menús con opciones por línea de comandos se usa `getopts` (tema _Scripts con opciones_).
