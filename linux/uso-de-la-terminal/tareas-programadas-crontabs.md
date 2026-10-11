---
icon: terminal
---

# Tareas programadas (Crontabs)

### cron y crontab

`cron` ejecuta tareas de forma recurrente según un horario definido.

```bash
crontab -l            # ver las tareas programadas del usuario actual
crontab -e              # editar las tareas programadas del usuario actual
crontab -r                # borrar TODAS las tareas del usuario actual (sin confirmación)
sudo crontab -u usuario -l   # ver las tareas de otro usuario (requiere privilegios)
```

#### Formato de una línea de crontab

```
*  *  *  *  *  comando_a_ejecutar
│  │  │  │  │
│  │  │  │  └── día de la semana (0-7, 0 y 7 son domingo)
│  │  │  └───── mes (1-12)
│  │  └──────── día del mes (1-31)
│  └─────────── hora (0-23)
└────────────── minuto (0-59)
```

#### Ejemplos

```bash
# Cada día a las 3:00 AM
0 3 * * * /home/pyrhos/scripts/backup.sh

# Cada 15 minutos
*/15 * * * * /home/pyrhos/scripts/check.sh

# Cada lunes a las 9:00
0 9 * * 1 /home/pyrhos/scripts/reporte_semanal.sh

# El día 1 de cada mes a medianoche
0 0 1 * * /home/pyrhos/scripts/mensual.sh

# De lunes a viernes a las 8:00
0 8 * * 1-5 /home/pyrhos/scripts/laborable.sh
```

{% hint style="warning" %}
Cron ejecuta los scripts con un entorno mínimo, sin cargar `.bashrc` ni heredar el `$PATH` de tu sesión interactiva. Usar siempre rutas absolutas dentro del script (`/usr/bin/python3` en vez de `python3`) y redirigir la salida a un log para poder depurar fallos (`>> /var/log/mi_tarea.log 2>&1`).
{% endhint %}

#### cron a nivel de sistema

```bash
/etc/crontab                  # crontab global del sistema (incluye columna de usuario)
/etc/cron.d/                    # ficheros de crontab adicionales, uno por servicio/paquete
/etc/cron.daily/                  # scripts ejecutados una vez al día
/etc/cron.hourly/                   # scripts ejecutados una vez a la hora
/etc/cron.weekly/                     # scripts ejecutados una vez a la semana
```

#### Atajos especiales

```
@reboot comando        # se ejecuta al arrancar el sistema
@daily comando            # equivalente a "0 0 * * *"
@weekly comando              # equivalente a "0 0 * * 0"
@monthly comando               # equivalente a "0 0 1 * *"
```

### at

A diferencia de cron (recurrente), `at` programa una ejecución única.

```bash
echo "comando" | at 15:30                 # hoy a las 15:30
at 15:30 -f script.sh                       # desde un fichero
echo "comando" | at now + 10 minutes          # dentro de 10 minutos
echo "comando" | at 9am tomorrow                # mañana a las 9

atq                    # ver tareas pendientes
atrm numero               # cancelar una tarea pendiente (el número lo da atq)
```

\{% hint style="info" %\} `at` puede no venir instalado por defecto: `sudo apt install at` y asegurarse de que el servicio `atd` está activo (`systemctl status atd`). \{% endhint %\}

### systemd timers

es la alternativa moderna a cron, un timer se compone de dos unit files: uno `.timer` (cuándo) y uno `.service` (qué).

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Timer para backup diario

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Backup diario

[Service]
Type=oneshot
ExecStart=/home/pyrhos/scripts/backup.sh
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer

systemctl list-timers          # ver todos los timers activos y su próxima ejecución
```

|                                     | cron                                  | systemd timer                                             |
| ----------------------------------- | ------------------------------------- | --------------------------------------------------------- |
| Sintaxis                            | Compacta, una línea                   | Más verbosa, dos ficheros                                 |
| Logging                             | Depende de cómo lo configures tú      | Integrado en `journalctl` automáticamente                 |
| Dependencias                        | No gestiona dependencias entre tareas | Puede depender de otros servicios/targets                 |
| Ejecución perdida (sistema apagado) | No se recupera                        | `Persistent=true` la ejecuta en cuanto el sistema arranca |
