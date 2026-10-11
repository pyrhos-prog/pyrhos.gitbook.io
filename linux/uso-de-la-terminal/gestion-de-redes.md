---
icon: terminal
---

# Gestión de redes

### Interfaces e IP

```bash
ip a                    # ver interfaces y direcciones IP (forma moderna)
ip addr show eth0         # solo una interfaz
ifconfig                  # forma clásica, puede no venir instalada por defecto

ip link set eth0 up        # levantar interfaz
ip link set eth0 down      # bajar interfaz
```

### Rutas

```bash
ip route                 # tabla de rutas
ip route show default      # solo la puerta de enlace por defecto
route -n                   # forma clásica
```

### Conexiones activas y puertos en escucha: ss / netstat

```bash
ss -tln              # puertos TCP en escucha (t=tcp, l=listen, n=numérico)
ss -tlnp               # igual, + proceso que lo escucha (necesita privilegios para verlo todo)
ss -tunap                # todas las conexiones TCP y UDP, con proceso
netstat -tlnp             # equivalente clásico, puede no estar instalado
```

#### Lectura rápida de `ss -tlnp`

```
State   Local Address:Port    Process
LISTEN  0.0.0.0:22            sshd
LISTEN  127.0.0.1:3306        mysqld
```

`0.0.0.0` = escucha en todas las interfaces. `127.0.0.1` = solo accesible localmente.

### ping y traceroute

```bash
ping -c 4 8.8.8.8            # 4 paquetes y para (sin -c, no para nunca)
ping -c 1 -W 1 host            # 1 paquete, timeout de 1 segundo — útil en scripts
traceroute 8.8.8.8               # ruta de saltos hasta el destino
mtr 8.8.8.8                        # traceroute + ping continuo combinados
```

### dig y nslookup

```bash
dig dominio.com                # consulta DNS completa
dig +short dominio.com          # solo la respuesta, sin cabecera
dig dominio.com MX               # tipo de registro concreto
dig -x 8.8.8.8                     # resolución inversa (IP a nombre)
nslookup dominio.com                 # alternativa más antigua
host dominio.com                       # forma rápida y simple
```

### curl y wget básicos

```bash
curl -I https://dominio.com          # solo cabeceras (HEAD)
curl -s https://dominio.com            # silencioso, sin barra de progreso
curl -s ifconfig.me                      # ver IP pública propia
wget https://dominio.com/fichero           # descargar un fichero
```

### hostname y resolución local

```bash
hostname           # nombre del equipo
hostname -I          # IPs asignadas, forma rápida
cat /etc/hosts         # resolución local de nombres
```

### Firewall: iptables, nftables y ufw

#### iptables (el clásico, aún muy presente)

```bash
iptables -L                       # listar reglas activas
iptables -L -v -n                   # con contadores y sin resolver DNS (más rápido y claro)
iptables -L -n --line-numbers         # con número de línea, útil para borrar reglas concretas

iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # permitir entrada por el puerto 22
iptables -A INPUT -s 192.168.1.10 -j DROP          # bloquear todo el tráfico de una IP
iptables -A INPUT -p tcp --dport 80 -j DROP          # bloquear el puerto 80
iptables -D INPUT 3                                    # borrar la regla número 3 de la cadena INPUT

iptables -P INPUT DROP             # política por defecto: descartar todo lo no permitido explícitamente
iptables -F                          # vaciar todas las reglas (flush) — cuidado, deja el firewall abierto
```

| Cadena    | Para qué tráfico                                 |
| --------- | ------------------------------------------------ |
| `INPUT`   | Tráfico entrante hacia esta máquina              |
| `OUTPUT`  | Tráfico saliente desde esta máquina              |
| `FORWARD` | Tráfico que atraviesa esta máquina (routing/NAT) |

```bash
# Guardar reglas para que sobrevivan a un reinicio (Debian/Ubuntu)
apt install iptables-persistent
netfilter-persistent save
```

#### nftables (sucesor moderno de iptables)

```bash
nft list ruleset                    # ver todas las reglas activas
nft list tables                       # listar tablas

nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; }
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input drop
```

\{% hint style="info" %\} `nftables` sustituye a `iptables` en distros modernas, pero la sintaxis de `iptables` se sigue viendo muchísimo en guías, CTFs y sistemas en producción. Vale la pena conocer ambos. \{% endhint %\}

#### ufw (Uncomplicated Firewall, capa simplificada sobre iptables)

```bash
ufw status                    # ver estado y reglas
ufw status verbose              # con más detalle

ufw enable                       # activar el firewall
ufw disable                        # desactivar

ufw allow 22                         # permitir un puerto
ufw allow 22/tcp                       # permitir un puerto + protocolo concreto
ufw allow from 192.168.1.10              # permitir todo el tráfico de una IP
ufw deny 80                                # bloquear un puerto
ufw delete allow 22                          # borrar una regla

ufw default deny incoming         # política por defecto: denegar entrante
ufw default allow outgoing          # política por defecto: permitir saliente
```

#### Comprobar qué está realmente filtrando el tráfico

```bash
# Ver si un puerto está realmente alcanzable desde fuera (no solo si el servicio escucha)
nmap -p 22 <tu_ip_publica>

# Ver reglas activas del kernel, independientemente de la herramienta usada para crearlas
iptables -L -n -v
```

***

### Escaneo rápido de puertos sin nmap

```bash
nc -zv 192.168.1.1 1-100      # netcat en modo escaneo (z=sin enviar datos, v=verbose)
timeout 1 bash -c "</dev/tcp/192.168.1.1/22" && echo "abierto"   # bash puro, sin herramientas extra
```

