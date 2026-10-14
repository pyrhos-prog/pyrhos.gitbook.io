# Señales y procesos

> Control del ciclo de vida de un script y de los procesos: captura de Ctrl+C con `trap`, señales, ejecución en segundo plano y en paralelo.

### Control de Ctrl+C con trap

Por defecto, `Ctrl+C` envía la señal `SIGINT` y mata el script en el punto en que esté, dejando la terminal en el estado que tuviera (cursor oculto, ficheros temporales sin borrar...). `trap` intercepta la señal y ejecuta una función de limpieza.

#### Sintaxis

```bash
trap nombre_funcion SEÑAL
trap 'comando1; comando2' SEÑAL
```

#### Ejemplo básico

```bash
#!/bin/bash

function ctrl_c(){
  echo -e "\n\n[!] Saliendo...\n"
  tput cnorm          # restaura el cursor
  exit 1
}

trap ctrl_c INT

tput civis            # oculta el cursor
while true; do
  sleep 1
done
```

#### Puntos clave

* `INT` es la señal de `Ctrl+C` (también válido `SIGINT` o el número `2`).
* El `trap` se registra al principio del script, antes del flujo principal.
* Dentro de la función se debe llamar a `exit`; si no, el script continúa tras la interrupción.
* `tput civis` oculta el cursor y `tput cnorm` lo restaura. Si se oculta, hay que restaurarlo siempre al salir.
* La función de limpieza es el lugar para borrar ficheros temporales o detener procesos lanzados por el script.

#### Limpieza completa

```bash
tmp=$(mktemp)

function ctrl_c(){
  echo -e "\n[!] Interrumpido"
  rm -f "$tmp"
  tput cnorm
  exit 130              # 128 + 2 (SIGINT)
}

trap ctrl_c INT
```

#### Ignorar y restaurar

```bash
trap '' INT             # ignorar Ctrl+C (no recomendado de forma general)
trap - INT              # restaurar el comportamiento por defecto
trap -p                 # listar los traps activos
```

### Señales y trap EXIT

Una señal es un aviso que el sistema envía a un proceso.

| Señal           | Número  | Origen / efecto                                                |
| --------------- | ------- | -------------------------------------------------------------- |
| `HUP`           | 1       | Cierre de la terminal; muchos servicios recargan configuración |
| `INT`           | 2       | Ctrl+C                                                         |
| `QUIT`          | 3       | Ctrl+\ (termina y genera volcado)                              |
| `KILL`          | 9       | Terminación forzosa; **no se puede capturar**                  |
| `USR1` / `USR2` | 10 / 12 | Definidas por la aplicación                                    |
| `TERM`          | 15      | Terminación ordenada (valor por defecto de `kill`)             |
| `CONT`          | 18      | Reanuda un proceso detenido                                    |
| `STOP`          | 19      | Detiene el proceso; **no se puede capturar**                   |
| `TSTP`          | 20      | Ctrl+Z (suspensión desde teclado)                              |

```bash
trap -l                 # lista de señales disponibles
```

Los números corresponden a Linux en arquitecturas habituales.

#### Varias señales

```bash
trap limpiar INT TERM HUP
```

#### trap EXIT

Se ejecuta **siempre** que el script termine: finalización normal, `exit`, error o señal capturada.

```bash
function limpiar(){
  rm -f "$tmp"
  tput cnorm
}

tmp=$(mktemp)
trap limpiar EXIT
```

Patrón recomendado: `EXIT` para la limpieza y `INT`/`TERM` solo para convertir la señal en un `exit`:

```bash
trap limpiar EXIT
trap 'exit 130' INT
trap 'exit 143' TERM
```

#### Otros traps

```bash
trap 'echo "Error en la línea $LINENO" >&2' ERR     # al fallar un comando (con set -e o -E)
trap 'echo "Recargando"' USR1                       # reaccionar a una señal personalizada
```

### Procesos: ps, top, kill

#### Listar procesos

```bash
ps aux                       # todos los procesos con detalle
ps -ef                       # formato alternativo, incluye PPID
ps aux | grep nginx          # buscar uno concreto
ps -p 1234 -o pid,ppid,cmd   # proceso concreto
pgrep -a nginx               # PIDs y comandos que coinciden con el nombre
pidof nginx                  # PIDs por nombre exacto
top                          # monitor interactivo (q para salir)
htop                         # versión mejorada, si está instalado
```

#### Enviar señales

```bash
kill 1234                    # SIGTERM (terminación ordenada)
kill -15 1234                # equivalente
kill -9 1234                 # SIGKILL (último recurso)
kill -HUP 1234               # otra señal por nombre
kill -l                      # lista de señales

pkill nginx                  # por nombre
pkill -f "script.sh"         # por línea de comandos completa
killall nginx                # todos los procesos con ese nombre
```

Se prueba primero `TERM` para que el proceso pueda cerrarse limpiamente y solo se usa `KILL` si no responde.

#### Comprobar si un proceso existe

```bash
if kill -0 "$pid" 2>/dev/null; then
  echo "El proceso $pid sigue vivo"
fi
```

#### Información en /proc

```bash
cat /proc/$$/cmdline         # línea de comandos del proceso actual
ls -l /proc/$$/fd            # descriptores abiertos
cat /proc/loadavg            # carga del sistema
```

#### Variables relacionadas

| Variable | Contenido                               |
| -------- | --------------------------------------- |
| `$$`     | PID del script                          |
| `$PPID`  | PID del proceso padre                   |
| `$!`     | PID del último proceso en segundo plano |

### Segundo plano: &, jobs, fg, bg

#### Lanzar en segundo plano

```bash
sleep 60 &                   # con & al final
echo "PID: $!"               # PID del proceso lanzado
```

#### Suspender y reanudar

```bash
sleep 300                    # proceso en primer plano
# Ctrl+Z -> queda detenido
jobs                         # lista de trabajos de la sesión
bg %1                        # reanuda el trabajo 1 en segundo plano
fg %1                        # lo trae a primer plano
kill %1                      # termina el trabajo 1
```

| Comando     | Efecto                                                     |
| ----------- | ---------------------------------------------------------- |
| `jobs`      | Lista los trabajos de la terminal actual                   |
| `fg %N`     | Trae el trabajo N a primer plano                           |
| `bg %N`     | Continúa el trabajo N en segundo plano                     |
| `disown %N` | Desvincula el trabajo de la terminal (sobrevive al cierre) |
| `wait`      | Espera a que terminen los procesos hijo                    |

#### Redirigir la salida del proceso en segundo plano

```bash
comando > salida.log 2>&1 &
```

Un proceso en segundo plano que escribe en la terminal interfiere con lo que se escribe después.

### wait y ejecución en paralelo

#### wait

```bash
comando_lento &
pid=$!
echo "Haciendo otras cosas..."
wait "$pid"                  # espera a ese proceso
echo "Código de salida: $?"
```

`wait` sin argumentos espera a todos los hijos.

#### Lanzar varias tareas en paralelo

```bash
function procesar(){
  echo "Inicio $1"
  sleep 2
  echo "Fin $1"
}

for i in 1 2 3 4; do
  procesar "$i" &
done
wait
echo "Todo terminado"
```

#### Limitar el número de tareas simultáneas

```bash
max=4
for i in {1..20}; do
  procesar "$i" &
  while (( $(jobs -r | wc -l) >= max )); do
    sleep 0.2
  done
done
wait
```

#### Con xargs -P

```bash
printf '%s\n' {1..20} | xargs -P 4 -I{} sh -c 'echo "Tarea {}"; sleep 1'
cat hosts.txt | xargs -P 8 -I{} ping -c1 -W1 {}
```

#### Recoger resultados de cada tarea

```bash
declare -A pids
for h in 192.0.2.1 192.0.2.2; do
  ping -c1 -W1 "$h" &>/dev/null &
  pids[$h]=$!
done

for h in "${!pids[@]}"; do
  if wait "${pids[$h]}"; then echo "$h responde"; else echo "$h no responde"; fi
done
```

### nohup, timeout y lockfiles

#### nohup

Evita que el proceso muera cuando se cierra la terminal (señal `HUP`).

```bash
nohup ./script_largo.sh > salida.log 2>&1 &
```

Si no se redirige, la salida se guarda en `nohup.out`. Otras opciones para sesiones persistentes: `tmux` y `screen`.

#### timeout

Limita el tiempo de ejecución de un comando.

```bash
timeout 5 ping 192.0.2.1             # termina a los 5 segundos
timeout -s KILL 10 comando           # con otra señal
timeout 3 bash -c 'echo > /dev/tcp/192.0.2.1/22' && echo "abierto"

timeout 5 comando
echo $?                              # 124 si se agotó el tiempo
```

#### Evitar ejecuciones simultáneas: lockfile

Con `flock` (recomendado, el bloqueo se libera solo al terminar):

```bash
exec 9> /tmp/mi_script.lock
if ! flock -n 9; then
  echo "Ya hay otra instancia en ejecución" >&2
  exit 1
fi

# ... resto del script ...
```

Con un directorio como cerrojo (`mkdir` es atómico):

```bash
lock="/tmp/mi_script.lock"
if ! mkdir "$lock" 2>/dev/null; then
  echo "Ya hay otra instancia en ejecución" >&2
  exit 1
fi
trap 'rmdir "$lock"' EXIT
```

#### Reintentos con espera

```bash
for intento in {1..5}; do
  if comando_inestable; then break; fi
  echo "Intento $intento fallido, reintentando..." >&2
  sleep $((intento * 2))
done
```

#### Ejecutar con prioridad baja

```bash
nice -n 10 ./tarea_pesada.sh
renice -n 10 -p 1234
```
