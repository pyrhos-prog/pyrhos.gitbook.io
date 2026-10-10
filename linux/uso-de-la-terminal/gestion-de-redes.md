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

### Escaneo rápido de puertos sin nmap

```bash
nc -zv 192.168.1.1 1-100      # netcat en modo escaneo (z=sin enviar datos, v=verbose)
timeout 1 bash -c "</dev/tcp/192.168.1.1/22" && echo "abierto"   # bash puro, sin herramientas extra
```
