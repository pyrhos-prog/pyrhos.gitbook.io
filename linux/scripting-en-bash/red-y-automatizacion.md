# Red y automatización

> Uso de Bash para interactuar con servicios de red, ejecutar tareas en equipos remotos y programar ejecuciones periódicas.

### curl y wget

#### curl

Transfiere datos con muchos protocolos; es la herramienta habitual para HTTP.

```bash
curl https://example.com                    # muestra el cuerpo
curl -s https://example.com                 # silencioso (sin barra de progreso)
curl -sS https://example.com                # silencioso, pero muestra errores
curl -o pagina.html https://example.com    # guarda con nombre
curl -O https://example.com/fichero.zip    # guarda con el nombre remoto
curl -L https://example.com                 # sigue redirecciones
curl -I https://example.com                 # solo cabeceras (HEAD)
curl -i https://example.com                 # cabeceras y cuerpo
curl -f https://example.com/noexiste        # falla (código distinto de 0) si HTTP >= 400
curl --max-time 10 https://example.com      # tiempo máximo total
```

#### Peticiones con datos

```bash
curl -X POST -d "usuario=ana&tema=bash" https://example.com/formulario

curl -X POST https://example.com/api \
  -H "Content-Type: application/json" \
  -d '{"clave": "valor"}'

curl -H "Authorization: Bearer $TOKEN" https://example.com/api/datos
curl -A "MiAgente/1.0" https://example.com           # cambiar User-Agent
curl -b "sesion=abc" -c cookies.txt https://example.com   # enviar y guardar cookies
```

Los tokens y contraseñas se leen de variables de entorno o ficheros protegidos, no se escriben en el script.

#### Obtener solo el código HTTP

```bash
codigo=$(curl -s -o /dev/null -w "%{http_code}" https://example.com)
echo "$codigo"
```

Otras variables de `-w`: `%{time_total}`, `%{size_download}`, `%{redirect_url}`.

#### Comprobar disponibilidad en un script

```bash
function web_ok(){
  [[ $(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$1") == 200 ]]
}

web_ok https://example.com && echo "Disponible" || echo "No disponible"
```

#### wget

Orientado a descargas.

```bash
wget https://example.com/fichero.zip
wget -q https://example.com/fichero.zip           # silencioso
wget -O nombre.zip https://example.com/f.zip      # nombre de salida
wget -c https://example.com/grande.iso            # reanuda una descarga interrumpida
wget -i urls.txt                                  # descarga una lista de URLs
wget -qO- https://example.com                     # imprime en stdout (como curl)
```

|                    | curl              | wget      |
| ------------------ | ----------------- | --------- |
| Uso principal      | Peticiones y APIs | Descargas |
| Salida por defecto | stdout            | Fichero   |
| Reanudar descargas | `-C -`            | `-c`      |
| Descarga recursiva | No                | Sí (`-r`) |

### Conexiones con /dev/tcp y nc

#### /dev/tcp

Bash puede abrir conexiones TCP sin herramientas externas (si no se compiló sin esa opción).

```bash
# Comprobar si un puerto está abierto
(echo > /dev/tcp/192.0.2.10/22) 2>/dev/null && echo "abierto" || echo "cerrado"

# Con tiempo máximo para evitar esperas largas
timeout 1 bash -c 'echo > /dev/tcp/192.0.2.10/22' 2>/dev/null && echo "abierto"
```

Enviar una petición HTTP a mano:

```bash
exec 3<> /dev/tcp/example.com/80
printf 'GET / HTTP/1.0\r\nHost: example.com\r\n\r\n' >&3
cat <&3
exec 3>&-
```

`/dev/tcp` es una característica de Bash, no un fichero real; no funciona con `ls` ni en otras shells.

#### nc (netcat)

```bash
nc -zv 192.0.2.10 22                 # comprueba un puerto (-z sin enviar datos, -v detallado)
nc -zv -w 1 192.0.2.10 20-25         # rango de puertos con 1 s de espera
nc -zvu 192.0.2.10 53                # UDP
```

Transferir un fichero entre dos equipos (la sintaxis exacta varía según la variante de `nc`):

```bash
# Receptor
nc -l 9000 > recibido.txt            # (algunas variantes requieren: nc -l -p 9000)

# Emisor
nc 192.0.2.10 9000 < fichero.txt
```

Consultar un servicio de texto:

```bash
printf 'HEAD / HTTP/1.0\r\nHost: example.com\r\n\r\n' | nc example.com 80
```

#### Otras herramientas de diagnóstico

```bash
ping -c 4 example.com                # conectividad
ss -tulpn                            # puertos en escucha (-t TCP, -u UDP, -l escucha, -p proceso, -n numérico)
ip a                                 # interfaces y direcciones
ip route                             # tabla de rutas
dig +short example.com               # resolución DNS
host example.com
traceroute example.com
```

### SSH y scp en scripts

#### Autenticación con claves

```bash
ssh-keygen -t ed25519 -C "comentario"      # genera el par de claves
ssh-copy-id usuario@192.0.2.10             # instala la clave pública en el servidor
ssh usuario@192.0.2.10                     # conexión sin contraseña
```

La clave privada (`~/.ssh/id_ed25519`) nunca se comparte ni se incluye en el script o en un repositorio. Debe tener permisos `600`.

#### Ejecutar comandos remotos

```bash
ssh usuario@192.0.2.10 'uptime'
ssh usuario@192.0.2.10 'df -h /; free -h'
ssh usuario@192.0.2.10 "ls $directorio_local"        # $directorio_local se expande AQUÍ
ssh usuario@192.0.2.10 'ls $HOME'                    # $HOME se expande en el servidor
```

#### Ejecutar un script local en el servidor remoto

```bash
ssh usuario@192.0.2.10 'bash -s' < script_local.sh
ssh usuario@192.0.2.10 'bash -s' -- arg1 arg2 < script_local.sh
```

#### Opciones útiles en automatización

```bash
ssh -o BatchMode=yes -o ConnectTimeout=5 usuario@192.0.2.10 'hostname'
```

| Opción             | Efecto                                                         |
| ------------------ | -------------------------------------------------------------- |
| `BatchMode=yes`    | No pregunta contraseñas ni confirmaciones; falla en su lugar   |
| `ConnectTimeout=N` | Tiempo máximo de conexión                                      |
| `-p PUERTO`        | Puerto distinto de 22                                          |
| `-i fichero`       | Clave privada concreta                                         |
| `-n`               | No lee stdin (necesario al usar ssh dentro de un `while read`) |
| `-T`               | Sin pseudo-terminal                                            |

#### Recorrer varios servidores

```bash
while IFS= read -r host; do
  echo "== $host =="
  ssh -n -o BatchMode=yes -o ConnectTimeout=5 "usuario@$host" 'uptime' || echo "Fallo en $host"
done < servidores.txt
```

#### Archivo de configuración

`~/.ssh/config` evita repetir parámetros:

```
Host servidor1
    HostName 192.0.2.10
    User usuario
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

```bash
ssh servidor1
```

#### scp y rsync

```bash
scp fichero.txt usuario@192.0.2.10:/tmp/           # subir
scp usuario@192.0.2.10:/var/log/app.log .          # descargar
scp -r carpeta usuario@192.0.2.10:/tmp/            # directorio
rsync -avz -e ssh carpeta/ usuario@192.0.2.10:/tmp/carpeta/
```

### Tareas programadas: cron y at

#### cron

Ejecuta comandos periódicamente.

```bash
crontab -e          # editar las tareas del usuario
crontab -l          # listar
crontab -r          # eliminar todas (sin confirmación)
```

#### Formato

```
┌──────── minuto (0-59)
│ ┌────── hora (0-23)
│ │ ┌──── día del mes (1-31)
│ │ │ ┌── mes (1-12)
│ │ │ │ ┌ día de la semana (0-7, 0 y 7 = domingo)
│ │ │ │ │
* * * * *  comando
```

| Expresión           | Significado                       |
| ------------------- | --------------------------------- |
| `* * * * *`         | Cada minuto                       |
| `*/5 * * * *`       | Cada 5 minutos                    |
| `0 * * * *`         | Cada hora en punto                |
| `30 2 * * *`        | Cada día a las 02:30              |
| `0 9 * * 1-5`       | De lunes a viernes a las 09:00    |
| `0 0 1 * *`         | El día 1 de cada mes a medianoche |
| `@reboot`           | Al arrancar el sistema            |
| `@daily`, `@weekly` | Atajos                            |

```
*/10 * * * * /home/usuario/scripts/monitor.sh >> /home/usuario/logs/monitor.log 2>&1
0 3 * * * /home/usuario/scripts/copia.sh
```

#### Problemas habituales

* El entorno de cron es mínimo: `PATH` reducido y sin variables de la sesión. Se usan rutas absolutas o se define `PATH` al inicio del crontab.
* Hay que redirigir la salida (`>> log 2>&1`); si no, cron intenta enviarla por correo.
* El script debe tener permiso de ejecución y shebang correcto.
* Los `%` en la línea de cron se interpretan como saltos de línea; se escapan como `\%`.
* Para depurar: `grep CRON /var/log/syslog` (o `journalctl -u cron`).

#### at

Para ejecutar una tarea **una sola vez**.

```bash
echo "/home/usuario/scripts/tarea.sh" | at 23:00
echo "reboot" | sudo at now + 1 hour
at -f tarea.sh tomorrow 09:00
atq                       # tareas pendientes
atrm 3                    # eliminar la tarea 3
```

### Servicios y timers con systemd

Alternativa moderna a cron, con registro integrado en el journal y gestión de dependencias.

#### Unidad de servicio

`/etc/systemd/system/copia.service`:

```ini
[Unit]
Description=Copia de seguridad

[Service]
Type=oneshot
ExecStart=/usr/local/bin/copia.sh
```

#### Timer asociado

`/etc/systemd/system/copia.timer`:

```ini
[Unit]
Description=Ejecuta la copia cada día

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

* `OnCalendar` define cuándo (`daily`, `weekly`, `*-*-* 03:00:00`, `Mon..Fri 09:00`).
* `Persistent=true` ejecuta la tarea al arrancar si se perdió una ejecución mientras el equipo estaba apagado.

#### Activar

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now copia.timer
systemctl list-timers
systemctl status copia.timer
```

#### Servicio permanente

Para un script que debe estar siempre en ejecución:

```ini
[Unit]
Description=Mi servicio
After=network.target

[Service]
ExecStart=/usr/local/bin/servicio.sh
Restart=on-failure
User=usuario

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now servicio.service
sudo systemctl restart servicio.service
journalctl -u servicio.service -f          # ver el log en tiempo real
journalctl -u copia.service --since today
```

#### cron frente a systemd timers

|                                    | cron                   | systemd timer          |
| ---------------------------------- | ---------------------- | ---------------------- |
| Configuración                      | Una línea              | Dos ficheros de unidad |
| Logs                               | Manual (redirecciones) | `journalctl`           |
| Ejecuciones perdidas               | No se recuperan        | `Persistent=true`      |
| Dependencias y límites de recursos | No                     | Sí                     |

### Copias de seguridad con rsync y tar

#### tar

```bash
tar -czf copia.tar.gz carpeta/          # crear (c), comprimir con gzip (z), fichero (f)
tar -xzf copia.tar.gz                   # extraer
tar -xzf copia.tar.gz -C /destino       # extraer en otro directorio
tar -tzf copia.tar.gz                   # listar contenido sin extraer
tar -cjf copia.tar.bz2 carpeta/         # bzip2
tar -cJf copia.tar.xz carpeta/          # xz
tar -czf copia.tar.gz --exclude='*.tmp' carpeta/
```

Nombre con fecha:

```bash
tar -czf "copia_$(date +%Y%m%d_%H%M%S).tar.gz" carpeta/
```

#### rsync

Sincroniza directorios copiando solo las diferencias.

```bash
rsync -avh origen/ destino/                   # modo archivo, detallado, legible
rsync -avh --delete origen/ destino/          # borra en destino lo que ya no está en origen
rsync -avhn origen/ destino/                  # -n: simulación, no hace cambios
rsync -avh --exclude='*.log' origen/ destino/
rsync -avz -e ssh origen/ usuario@192.0.2.10:/copias/
rsync -avh --progress origen/ destino/
```

La barra final importa:

* `origen/` copia el **contenido** del directorio.
* `origen` copia el directorio mismo dentro del destino.

Con `--delete` se recomienda ejecutar antes una simulación con `-n`.

#### Rotación de copias

Conservar solo las N más recientes:

```bash
retencion=7
ls -1t /copias/copia_*.tar.gz | tail -n +$((retencion + 1)) | xargs -r rm --
```

#### Verificar integridad

```bash
sha256sum copia.tar.gz > copia.tar.gz.sha256
sha256sum -c copia.tar.gz.sha256
tar -tzf copia.tar.gz > /dev/null && echo "Archivo íntegro"
```

#### Copias incrementales con enlaces duros

```bash
rsync -a --link-dest=/copias/anterior /datos/ "/copias/$(date +%F)/"
```

Cada copia aparece completa, pero los ficheros sin cambios comparten espacio en disco con la anterior.
