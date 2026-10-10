---
icon: terminal
---

# Procesos y trabajos

### ps

```bash
ps                 # procesos de la terminal actual
ps aux              # TODOS los procesos del sistema, formato detallado
ps -ef              # alternativa, formato distinto (estilo System V)
ps aux | grep nginx  # buscar un proceso concreto
```

#### Lectura de `ps aux`

```
USER   PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
root     1   0.0  0.1  16928  9876 ?     Ss   09:00   0:02 /sbin/init
```

| Campo           | Significado                                                    |
| --------------- | -------------------------------------------------------------- |
| `USER`          | Propietario del proceso                                        |
| `PID`           | ID del proceso                                                 |
| `%CPU` / `%MEM` | Uso de CPU/memoria                                             |
| `STAT`          | Estado (`R` ejecutando, `S` durmiendo, `Z` zombie, `T` parado) |
| `TTY`           | Terminal asociada (`?` si no tiene)                            |
| `COMMAND`       | Comando que lo lanzó                                           |

```bash
ps aux --sort=-%cpu | head     # top 10 por uso de CPU
ps aux --sort=-%mem | head     # top 10 por uso de memoria
ps -p 1234                     # info de un PID concreto
pstree -p                      # árbol de procesos padre/hijo, con PIDs
```

### top / htop

```bash
top          # monitor interactivo en tiempo real
htop         # versión mejorada (colores, scroll, más intuitivo)
```

| Tecla (dentro de `top`) | Acción                      |
| ----------------------- | --------------------------- |
| `q`                     | Salir                       |
| `k`                     | Matar un proceso (pide PID) |
| `M`                     | Ordenar por memoria         |
| `P`                     | Ordenar por CPU             |
| `r`                     | Cambiar prioridad (renice)  |
| `1`                     | Mostrar uso por cada núcleo |

### kill, killall y señales

`kill` no mata directamente: envía una señal al proceso, y es el proceso (o el kernel) quien decide qué hacer con ella.

```bash
kill PID                  # envía SIGTERM (petición de cierre ordenado)
kill -9 PID                 # envía SIGKILL (terminación forzosa, inmediata)
killall nombre_proceso        # mata por nombre, todas las instancias
pkill -f "patrón"              # mata procesos cuyo comando completo coincide con el patrón
```

| Señal     | Número | Efecto                                    |
| --------- | ------ | ----------------------------------------- |
| `SIGHUP`  | 1      | Recarga config / cierre de terminal       |
| `SIGINT`  | 2      | Interrupción (Ctrl+C)                     |
| `SIGKILL` | 9      | Terminación forzosa, no se puede ignorar  |
| `SIGTERM` | 15     | Petición de cierre ordenado (por defecto) |
| `SIGSTOP` | 19     | Pausa el proceso                          |
| `SIGCONT` | 18     | Reanuda un proceso parado                 |

```bash
kill -l              # lista todas las señales disponibles
pgrep nombre_proceso    # busca PID por nombre, sin ps+grep
```

{% hint style="info" %}
&#x20;`SIGKILL` (-9) no da al proceso oportunidad de cerrar ficheros ni liberar recursos. Úsalo solo si `SIGTERM` no ha funcionado.&#x20;
{% endhint %}

### Foreground / background: jobs, fg, bg

```bash
comando &              # lanza en segundo plano
nmap -sS 10.0.0.0/24 -oN scan.txt &

jobs                     # lista los trabajos de la sesión actual
jobs -l                    # igual, con PID
```

```
[1]-  Running    nmap -sS 10.0.0.0/24 &
[2]+  Stopped    vim fichero.txt
```

`Ctrl+Z` suspende el proceso en foreground y lo deja parado en background.

```bash
fg               # trae el job actual al primer plano
fg %2              # trae el job número 2
bg                 # reanuda el job actual en background (tras un Ctrl+Z)
kill %1              # mata el job 1 (no el PID directamente)
```

### nohup y disown

Si cierras la terminal, los procesos en background normalmente reciben `SIGHUP` y mueren con ella.

```bash
nohup comando &                      # ignora SIGHUP, sigue vivo si cierras la terminal
nohup comando > salida.log 2>&1 &     # además, redirige toda la salida a un log

disown               # desvincula el job actual de la shell (sigue corriendo tras cerrar)
disown -a               # desvincula todos los jobs
```

{% hint style="info" %}
Para procesos que necesitas poder recuperar después (no solo que sobrevivan), mejor usar `tmux`/`screen` — puedes volver a "ver" la sesión completa, no solo que el proceso siga vivo.

&#x20;Ver [13 — tmux y screen](https://claude.ai/chat/13-tmux-screen.md).
{% endhint %}

### systemctl básico

Gestión de servicios en sistemas con systemd.

```bash
systemctl status nginx         # estado de un servicio
systemctl start nginx           # arrancar
systemctl stop nginx             # parar
systemctl restart nginx           # reiniciar
systemctl reload nginx             # recargar config sin cortar conexiones
systemctl enable nginx              # activar en el arranque del sistema
systemctl disable nginx              # desactivar en el arranque
systemctl is-enabled nginx            # comprobar si está activado en el arranque
journalctl -u nginx -f                # logs del servicio en tiempo real
```
