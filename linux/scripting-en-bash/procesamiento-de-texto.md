# Procesamiento de texto

> Herramientas para filtrar, transformar y resumir texto desde la línea de comandos. Se combinan con pipes para construir cadenas de procesado.

### Visualización de ficheros

| Comando       | Uso                                             |
| ------------- | ----------------------------------------------- |
| `cat f`       | Muestra el fichero completo                     |
| `cat -n f`    | Con números de línea                            |
| `tac f`       | Muestra las líneas en orden inverso             |
| `head -n 5 f` | Primeras 5 líneas                               |
| `tail -n 5 f` | Últimas 5 líneas                                |
| `tail -f f`   | Sigue el fichero en tiempo real (útil con logs) |
| `less f`      | Paginador (`/texto` busca, `q` sale)            |
| `wc -l f`     | Número de líneas                                |
| `nl f`        | Numera las líneas                               |

```bash
head -n 20 fichero.txt | tail -n 5      # líneas 16 a 20
tail -n +10 fichero.txt                 # desde la línea 10 hasta el final
tail -f /var/log/syslog                 # seguir un log
```

### 2. grep y expresiones regulares

`grep` imprime las líneas que coinciden con un patrón.

```bash
grep "error" log.txt
grep -i "error" log.txt          # ignora mayúsculas
grep -v "debug" log.txt          # líneas que NO coinciden
grep -n "error" log.txt          # con número de línea
grep -c "error" log.txt          # cuenta las líneas que coinciden
grep -r "TODO" ./proyecto        # búsqueda recursiva
grep -w "root" /etc/passwd       # palabra completa
grep -o "[0-9]\+" log.txt        # solo la parte que coincide
grep -E "error|fallo" log.txt    # expresión regular extendida
grep -F "a.b" log.txt            # texto literal (sin regex)
grep -q "root" /etc/passwd       # silencioso: solo devuelve el código de salida
grep -l "root" /etc/*            # solo nombres de fichero con coincidencia
grep -A 2 -B 1 "error" log.txt   # 2 líneas después y 1 antes (-C n: ambas)
```

#### Expresiones regulares básicas

| Patrón                  | Significado              |
| ----------------------- | ------------------------ |
| `.`                     | Cualquier carácter       |
| `^`                     | Inicio de línea          |
| `$`                     | Final de línea           |
| `[abc]`                 | Uno de los caracteres    |
| `[^abc]`                | Cualquiera menos esos    |
| `[0-9]`                 | Rango                    |
| `*`                     | Cero o más repeticiones  |
| `\+` o `+` (con `-E`)   | Una o más                |
| `\?` o `?` (con `-E`)   | Cero o una               |
| `{n,m}` (con `-E`)      | Entre n y m repeticiones |
| `( )` y `\|` (con `-E`) | Agrupación y alternativa |
| `\b`                    | Límite de palabra        |

Clases POSIX: `[[:digit:]]`, `[[:alpha:]]`, `[[:space:]]`, `[[:upper:]]`.

```bash
grep -E "^[0-9]{3}-[0-9]{4}$" telefonos.txt      # formato 123-4567
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" log.txt    # posibles direcciones IPv4
grep "^#" config.conf                            # líneas que empiezan por #
grep -v "^#" config.conf | grep -v "^$"          # quitar comentarios y líneas vacías
```

`grep -P` activa expresiones compatibles con Perl (`\d`, `\K`, lookahead) cuando está disponible:

```bash
grep -oP "puerto=\K\d+" config.conf
```

### 3. cut, tr y paste

#### cut

Extrae columnas o caracteres.

```bash
cut -d: -f1 /etc/passwd              # campo 1 con delimitador :
cut -d: -f1,3 /etc/passwd            # campos 1 y 3
cut -d, -f2- datos.csv               # del campo 2 hasta el final
cut -c1-5 fichero.txt                # caracteres 1 a 5
```

`cut` solo admite un delimitador de un carácter; para espacios múltiples es más cómodo `awk`.

#### tr

Traduce, elimina o comprime caracteres (trabaja con stdin).

```bash
echo "hola" | tr 'a-z' 'A-Z'         # HOLA
echo "a  b   c" | tr -s ' '          # a b c   (comprime repetidos)
echo "a1b2" | tr -d '0-9'            # ab      (elimina)
echo "a1b2" | tr -cd '0-9'           # 12      (conserva solo dígitos)
cat fichero | tr '\n' ' '            # une líneas con espacio
echo "hola" | tr 'a-zA-Z' 'n-za-mN-ZA-M'   # ROT13
```

#### paste

Une ficheros línea a línea o convierte líneas en una sola.

```bash
paste a.txt b.txt                    # columnas separadas por tabulador
paste -d, a.txt b.txt                # separador coma
paste -sd, lista.txt                 # todas las líneas en una, separadas por comas
```

### 4. sort, uniq y wc

#### sort

```bash
sort fichero.txt
sort -r fichero.txt                  # inverso
sort -n numeros.txt                  # orden numérico
sort -u fichero.txt                  # elimina duplicados
sort -t: -k3 -n /etc/passwd          # por el campo 3, numérico, delimitador :
sort -h tamanos.txt                  # tamaños legibles (1K, 2M)
sort -V versiones.txt                # orden por versión (1.2, 1.10)
```

#### uniq

Elimina o cuenta **líneas consecutivas** repetidas; por eso suele ir después de `sort`.

```bash
sort fichero.txt | uniq              # sin duplicados
sort fichero.txt | uniq -c           # cuenta cuántas veces aparece cada línea
sort fichero.txt | uniq -d           # solo las repetidas
sort fichero.txt | uniq -u           # solo las que aparecen una vez
```

#### Patrón clásico: ranking de frecuencias

```bash
cat acceso.log | cut -d' ' -f1 | sort | uniq -c | sort -rn | head -n 10
```

#### wc

```bash
wc -l fichero.txt                    # líneas
wc -w fichero.txt                    # palabras
wc -c fichero.txt                    # bytes
```

### 5. sed

Editor de flujo: aplica transformaciones línea a línea.

#### Sustituir

```bash
sed 's/antiguo/nuevo/' f             # primera coincidencia de cada línea
sed 's/antiguo/nuevo/g' f            # todas las coincidencias
sed 's/antiguo/nuevo/gi' f           # ignorando mayúsculas
sed -E 's/([0-9]+)-([0-9]+)/\2-\1/' f   # grupos con regex extendida
sed 's|/usr/bin|/opt/bin|g' f        # otro delimitador para evitar escapar /
```

#### Borrar e imprimir

```bash
sed '3d' f                           # borra la línea 3
sed '2,5d' f                         # borra de la 2 a la 5
sed '/^#/d' f                        # borra líneas que empiezan por #
sed '/^$/d' f                        # borra líneas vacías
sed -n '5p' f                        # imprime solo la línea 5
sed -n '5,10p' f                     # imprime de la 5 a la 10
sed -n '/inicio/,/fin/p' f           # imprime entre dos patrones
```

#### Insertar y añadir

```bash
sed '1i Cabecera' f                  # inserta antes de la línea 1
sed '$a Pie' f                       # añade tras la última línea
```

#### Editar el fichero en el sitio

```bash
sed -i 's/antiguo/nuevo/g' f         # modifica el fichero
sed -i.bak 's/antiguo/nuevo/g' f     # y guarda una copia .bak
```

Conviene probar primero sin `-i` y comprobar el resultado.

### awk

Lenguaje orientado a columnas. Estructura: `patrón { acción }`.

```bash
awk '{print $1}' f                   # primera columna (separador: espacios)
awk '{print $1, $3}' f               # columnas 1 y 3
awk '{print $NF}' f                  # última columna
awk -F: '{print $1}' /etc/passwd     # separador personalizado
awk -F, -v OFS=';' '{print $1,$2}' f # cambia el separador de salida
```

#### Variables internas

| Variable      | Significado                   |
| ------------- | ----------------------------- |
| `$0`          | Línea completa                |
| `$1`, `$2`... | Columnas                      |
| `NF`          | Número de campos de la línea  |
| `NR`          | Número de línea actual        |
| `FS` / `OFS`  | Separador de entrada / salida |

#### Filtrar

```bash
awk 'NR > 1' f                       # omite la cabecera
awk '$3 > 100' f                     # columna 3 mayor que 100
awk '/error/' f                      # líneas que contienen "error"
awk '$1 == "root"' f
awk -F: '$3 >= 1000 {print $1}' /etc/passwd   # usuarios con UID >= 1000
```

#### BEGIN y END

```bash
awk 'BEGIN {print "Inicio"} {print $1} END {print "Fin"}' f
awk '{suma += $2} END {print suma}' f               # suma de la columna 2
awk '{suma += $2} END {print suma/NR}' f            # media
awk '{c[$1]++} END {for (k in c) print k, c[k]}' f  # contar por clave
```

#### printf

```bash
awk '{printf "%-10s %5d\n", $1, $2}' f
```

### find y xargs

#### find

```bash
find . -name "*.txt"                 # por nombre
find . -iname "*.txt"                # ignorando mayúsculas
find /var/log -type f                # solo ficheros (d = directorios, l = enlaces)
find . -type f -size +10M            # mayores de 10 MB
find . -type f -mtime -7             # modificados en los últimos 7 días
find . -type f -mmin -30             # en los últimos 30 minutos
find . -type f -perm 644             # con permisos concretos
find . -user usuario                 # por propietario
find . -maxdepth 2 -name "*.sh"      # limitar profundidad
find . -type f -empty                # vacíos
```

Combinar condiciones: `-and` (implícito), `-or`, `-not`.

```bash
find . -type f \( -name "*.jpg" -o -name "*.png" \)
```

#### Ejecutar acciones

```bash
find . -name "*.tmp" -delete
find . -name "*.sh" -exec chmod +x {} \;     # un comando por fichero
find . -name "*.log" -exec gzip {} +         # agrupa ficheros en una sola llamada
```

#### xargs

Convierte la entrada estándar en argumentos de otro comando.

```bash
find . -name "*.txt" | xargs wc -l
cat urls.txt | xargs -n 1 curl -sI           # un argumento por ejecución
echo "a b c" | xargs -I{} echo "Elemento: {}"
cat hosts.txt | xargs -P 4 -I{} ping -c1 {}  # 4 procesos en paralelo
```

Para nombres con espacios, usar separador nulo:

```bash
find . -name "*.txt" -print0 | xargs -0 rm
```

| Opción | Efecto                                |
| ------ | ------------------------------------- |
| `-n N` | N argumentos por ejecución            |
| `-I{}` | Marcador donde insertar cada elemento |
| `-P N` | Hasta N procesos simultáneos          |
| `-0`   | Entrada separada por caracteres nulos |
| `-r`   | No ejecutar si la entrada está vacía  |

### jq para JSON

`jq` filtra y transforma JSON. No viene instalado por defecto (`sudo apt install jq`).

Dado `datos.json`:

```json
{"nombre": "servidor1", "puertos": [22, 80], "activo": true}
```

```bash
jq '.' datos.json                    # formatea
jq '.nombre' datos.json              # "servidor1"
jq -r '.nombre' datos.json           # servidor1 (sin comillas)
jq '.puertos[0]' datos.json          # 22
jq '.puertos[]' datos.json           # un valor por línea
jq '.puertos | length' datos.json    # 2
jq 'select(.activo == true)' datos.json
```

Sobre listas de objetos:

```bash
jq '.[] | .nombre' lista.json
jq -r '.[] | "\(.nombre): \(.puerto)"' lista.json
jq '[.[] | select(.puerto > 1024)]' lista.json
```

Con APIs:

```bash
curl -s https://example.com/api/datos | jq -r '.resultado[].id'
```

Opciones útiles: `-r` salida sin comillas, `-c` salida compacta (un objeto por línea), `-e` devuelve código de salida según el resultado.
