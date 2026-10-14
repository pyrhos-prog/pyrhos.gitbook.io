# Entrada y salida

> Cómo se mueven los datos entre comandos, ficheros y el usuario: redirecciones, descriptores, pipes, here-documents, subshells.

### Redirecciones

Cada comando dispone de tres canales estándar:

| Descriptor | Nombre           | Por defecto |
| ---------- | ---------------- | ----------- |
| `0`        | stdin (entrada)  | Teclado     |
| `1`        | stdout (salida)  | Pantalla    |
| `2`        | stderr (errores) | Pantalla    |

#### Operadores

| Operador      | Efecto                                        |
| ------------- | --------------------------------------------- |
| `> fichero`   | Redirige stdout a un fichero (lo sobrescribe) |
| `>> fichero`  | Redirige stdout añadiendo al final            |
| `< fichero`   | Usa el fichero como stdin                     |
| `2> fichero`  | Redirige stderr                               |
| `2>> fichero` | Redirige stderr añadiendo                     |
| `&> fichero`  | Redirige stdout y stderr juntos (Bash)        |
| `2>&1`        | Envía stderr al mismo sitio que stdout        |
| `>&2`         | Envía stdout a stderr                         |

```bash
ls /etc > listado.txt             # guarda la salida
ls /etc >> listado.txt            # añade
ls /noexiste 2> errores.txt       # solo errores
ls /etc /noexiste > ok.txt 2> err.txt   # separados
ls /etc /noexiste &> todo.txt     # ambos en el mismo fichero
sort < desordenado.txt            # entrada desde fichero
```

#### Orden de 2>&1

El orden importa: se lee de izquierda a derecha.

```bash
comando > salida.txt 2>&1     # correcto: ambos van al fichero
comando 2>&1 > salida.txt     # stderr va a la pantalla, solo stdout al fichero
```

#### Mensajes de error a stderr

```bash
echo "Error: fichero no encontrado" >&2
```

#### Proteger ficheros de sobrescritura

```bash
set -o noclobber       # > no sobrescribe ficheros existentes
comando >| fichero     # fuerza la sobrescritura
```

#### Vaciar o crear un fichero

```bash
> fichero.txt          # lo deja vacío (o lo crea)
```

### Descriptores de fichero

Además de 0, 1 y 2, Bash permite abrir descriptores propios (3 en adelante) con `exec`.

```bash
exec 3> salida.log         # abre el descriptor 3 para escritura
echo "Primera línea" >&3
echo "Segunda línea" >&3
exec 3>&-                  # cierra el descriptor 3
```

Lectura:

```bash
exec 4< datos.txt
read -r primera <&4
read -r segunda <&4
exec 4<&-
```

#### Redirigir todo el script

```bash
exec > salida.log 2>&1     # desde este punto, toda la salida del script va al fichero
```

#### Duplicar descriptores

```bash
exec 5>&1                  # guarda el stdout original en el descriptor 5
exec > fichero.txt         # redirige stdout
echo "Va al fichero"
exec 1>&5 5>&-             # restaura stdout y cierra el 5
```

#### Descriptores de red con /dev/tcp

```bash
exec 3<> /dev/tcp/example.com/80
printf 'HEAD / HTTP/1.0\r\nHost: example.com\r\n\r\n' >&3
cat <&3
exec 3>&-
```

`<>` abre el descriptor en lectura y escritura.

### Pipes

El operador `|` conecta la stdout de un comando con la stdin del siguiente.

```bash
cat /etc/passwd | cut -d: -f1 | sort | head -n 5
ps aux | grep ssh
ls -l | wc -l
```

#### Incluir stderr en la tubería

```bash
comando 2>&1 | grep "error"
comando |& grep "error"        # forma abreviada de Bash
```

#### Códigos de salida en una tubería

Por defecto, el código de salida es el del **último** comando. Un fallo anterior pasa desapercibido.

```bash
false | true
echo $?                        # 0

set -o pipefail                # ahora la tubería falla si falla cualquier parte
false | true
echo $?                        # 1

echo "${PIPESTATUS[@]}"        # códigos de cada comando de la última tubería
```

#### Cada comando de la tubería se ejecuta en una subshell

Las variables modificadas dentro de la tubería no se ven fuera.

```bash
contador=0
echo -e "a\nb" | while read -r l; do ((contador++)); done
echo "$contador"               # 0
```

Solución: usar redirección o process substitution (ver más abajo).

### Here-doc y here-string

#### Here-document

Pasa un bloque de texto de varias líneas como entrada.

```bash
cat <<EOF
Línea 1
Usuario: $USER
Fecha: $(date +%F)
EOF
```

| Variante  | Efecto                                                         |
| --------- | -------------------------------------------------------------- |
| `<<EOF`   | Expande variables y comandos                                   |
| `<<'EOF'` | Texto literal, sin expansión                                   |
| `<<-EOF`  | Elimina los tabuladores iniciales (permite indentar el bloque) |

```bash
cat <<'EOF'
El símbolo $ y los comandos $(date) se muestran tal cual.
EOF
```

Crear un fichero desde un script:

```bash
cat > config.conf <<EOF
servidor=example.com
puerto=8080
EOF
```

Panel de ayuda:

```bash
function panel_ayuda(){
  cat <<EOF

Uso: $0 [opciones]
  -m  modo
  -h  mostrar esta ayuda

EOF
}
```

El delimitador de cierre (`EOF`) debe estar solo en su línea, sin espacios delante (salvo con `<<-` y tabuladores).

#### Here-string

Pasa una cadena como stdin.

```bash
grep "root" <<< "$texto"
read -r a b <<< "uno dos"
bc <<< "2 + 3"
```

### /dev/null y tee

#### /dev/null

Dispositivo que descarta todo lo que se escribe en él.

```bash
comando > /dev/null           # oculta la salida normal
comando 2> /dev/null          # oculta los errores
comando &> /dev/null          # oculta todo
```

Uso típico: comprobar si algo funciona sin mostrar nada.

```bash
if ping -c1 -W1 192.0.2.1 &> /dev/null; then
  echo "Responde"
fi
```

`/dev/zero` produce bytes a cero; `/dev/urandom` produce bytes aleatorios.

#### tee

Escribe la entrada en un fichero **y** la deja pasar por la salida estándar.

```bash
comando | tee salida.txt          # muestra y guarda
comando | tee -a salida.txt       # añade al final
comando 2>&1 | tee log.txt        # incluye errores
```

Escribir en un fichero que requiere privilegios (la redirección la hace el usuario, no `sudo`):

```bash
echo "linea" | sudo tee -a /etc/fichero.conf > /dev/null
```

### Process substitution

Permite usar la salida de un comando como si fuera un fichero.

```bash
diff <(sort a.txt) <(sort b.txt)       # compara dos salidas
comm -12 <(sort a.txt) <(sort b.txt)   # líneas comunes
```

* `<(comando)`: el resultado se lee como un fichero.
* `>(comando)`: lo escrito se envía como entrada de un comando.

```bash
comando | tee >(grep error > errores.txt) > completo.txt
```

#### Evitar la subshell en un while

```bash
contador=0
while read -r linea; do
  ((contador++))
done < <(ls /etc)
echo "$contador"                       # el valor se conserva
```

Process substitution es una característica de Bash, no existe en `sh` POSIX.

### Subshells y agrupación de comandos

#### Subshell: ( ... )

Ejecuta los comandos en un proceso hijo. Los cambios de variables y de directorio no afectan a la shell actual.

```bash
pwd
( cd /tmp && pwd )
pwd                       # sigue en el directorio original

x=1
( x=2; echo "$x" )        # 2
echo "$x"                 # 1
```

#### Agrupación: { ...; }

Ejecuta los comandos en la shell actual y permite redirigirlos en conjunto.

```bash
{ echo "Cabecera"; date; ls; } > informe.txt

[[ -f "$f" ]] || { echo "Falta $f" >&2; exit 1; }
```

Sintaxis estricta: espacio después de `{` y `;` (o salto de línea) antes de `}`.

|                  | `( )`          | `{ }`                              |
| ---------------- | -------------- | ---------------------------------- |
| Proceso          | Subshell       | Shell actual                       |
| Variables y `cd` | No persisten   | Persisten                          |
| Uso típico       | Aislar cambios | Agrupar para redirigir o encadenar |

#### Sustitución de comandos

`$( )` también ejecuta en una subshell y captura su salida.

```bash
fecha=$(date +%F)
ficheros=$(ls | wc -l)
```

#### Segundo plano

```bash
( sleep 5; echo "Terminado" ) &
```
