---
icon: webhook
---

# OWASP API Security Top 10

### ¿Qué es el OWASP API Security Top 10?

El OWASP API Security Top 10 es una lista independiente del OWASP Web Top 10, publicada específicamente para APIs REST, GraphQL, SOAP y WebSockets. La versión más reciente es de 2023.

Las APIs tienen superficies de ataque distintas a las aplicaciones web tradicionales: exponen lógica de negocio directamente, devuelven datos en formato estructurado (JSON/XML), tienen autenticación basada en tokens y no en sesiones, y suelen estar menos protegidas que el frontend.

```
Aplicación web tradicional:
Usuario → Navegador → HTML renderizado → Servidor

API moderna:
App móvil    ┐
SPA React    ├──→ API REST/GraphQL → Servidor
Microservicio┘
```

### Diferencias clave con el OWASP Web Top 10

| OWASP Web Top 10           | OWASP API Top 10                     |
| -------------------------- | ------------------------------------ |
| Aplicaciones web completas | Solo APIs                            |
| HTML + formularios         | JSON/XML + tokens                    |
| Sesiones con cookies       | JWT, API keys, OAuth                 |
| Errores visibles en UI     | Errores en respuestas JSON           |
| 1 usuario = 1 sesión       | 1 token = N peticiones automatizadas |
| SQLi, XSS frecuentes       | BOLA, mass assignment frecuentes     |

### Las 10 categorías (2023)

| #         | Nombre                                          | Descripción corta                                            |
| --------- | ----------------------------------------------- | ------------------------------------------------------------ |
| **API1**  | Broken Object Level Authorization (BOLA)        | Acceder a objetos de otros usuarios cambiando un ID          |
| **API2**  | Broken Authentication                           | Tokens débiles, JWT inseguros, API keys expuestas            |
| **API3**  | Broken Object Property Level Authorization      | Mass assignment + datos excesivos en respuestas              |
| **API4**  | Unrestricted Resource Consumption               | Sin rate limiting → DoS, scraping, abuso de costes           |
| **API5**  | Broken Function Level Authorization             | Acceder a funciones admin como usuario normal                |
| **API6**  | Unrestricted Access to Sensitive Business Flows | Abusar flujos de negocio a escala automatizada               |
| **API7**  | Server-Side Request Forgery                     | El servidor hace peticiones a URLs controladas               |
| **API8**  | Security Misconfiguration                       | CORS, métodos innecesarios, verbose errors                   |
| **API9**  | Improper Inventory Management                   | APIs antiguas expuestas, shadow APIs, falta de documentación |
| **API10** | Unsafe Consumption of APIs                      | Confiar sin validar en APIs de terceros                      |

### Por qué las APIs son más vulnerables

```
1. Exponen lógica directamente
   → GET /api/users/1042 devuelve el objeto completo
   → No hay capa de presentación que filtre los datos

2. Diseñadas para automatización
   → Un atacante puede hacer 10.000 peticiones fácilmente
   → Rate limiting ausente o mal configurado

3. Múltiples versiones coexisten
   → /api/v1/ puede seguir activa aunque /api/v3/ sea la oficial
   → v1 tiene controles de seguridad peores (fue la primera)

4. Documentación pública
   → Swagger/OpenAPI expone todos los endpoints automáticamente
   → Un atacante sabe exactamente qué existe sin tener que descubrirlo

5. Tokens = más poder que cookies
   → Una cookie robada da sesión de 1 usuario
   → Un token de API key da acceso ilimitado al 100% de la API

6. Errores más informativos
   → {"error": "user_id 1043 not found in organization 5"} → enumeration
   → Las APIs devuelven mensajes técnicos detallados por diseño
```

### Reconocimiento de APIs — antes de atacar

#### Descubrir la API

```bash
# Buscar en el JS de la app
# En DevTools → Sources → buscar /api/, fetch(, axios., XMLHttpRequest
grep -r "api/" --include="*.js" .
grep -r "fetch(" --include="*.js" .

# Buscar en el tráfico de red
# DevTools → Network → filtrar por Fetch/XHR → ver todas las llamadas a la API

# Fuzzing de rutas de API
ffuf -u https://target.com/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common-api-endpoints-mazen160.txt

# Kiterunner — descubrimiento de endpoints de API con rutas completas
kr scan https://target.com -w routes-large.kite
kr scan https://target.com -w /usr/share/seclists/Discovery/Web-Content/common-api-endpoints-mazen160.txt
```

#### Encontrar documentación expuesta

```bash
# Swagger / OpenAPI
/swagger.json
/swagger.yaml
/swagger-ui.html
/swagger-ui/
/api-docs
/api-docs.json
/api/swagger.json
/api/v1/swagger.json
/openapi.json
/openapi.yaml
/v1/api-docs
/v2/api-docs
/v3/api-docs

# Postman / API Collections
/postman_collection.json
/.env  # A veces contiene URLs de API y tokens

# GraphQL
/graphql
/api/graphql
/gql
/query

# WSDL (SOAP)
/wsdl
/api?wsdl
/service?wsdl
```

#### Herramientas de reconocimiento de APIs

```bash
# Arjun — descubrir parámetros ocultos en APIs
pip3 install arjun
arjun -u https://target.com/api/users
arjun -u https://target.com/api/search --get
arjun -u https://target.com/api/update --post

# Kiterunner — bruteforce de rutas de API
go install github.com/assetnote/kiterunner/cmd/kr@latest
kr scan https://target.com -w routes-large.kite -x 20

# Postman — importar collection y explorar
# Si se encuentra una collection de Postman expuesta:
# → Importar en Postman → ver todos los endpoints con sus parámetros
# → Buscar tokens hardcodeados en los ejemplos

# mitmproxy / Burp — interceptar tráfico de apps móviles
# Apps móviles consumen la misma API que el navegador
# Capturar tráfico de la app = descubrir endpoints no documentados
```

### Metodología de pentest de API

```
1. Reconocimiento
   → Encontrar la documentación (Swagger, OpenAPI, Postman)
   → Mapear todos los endpoints y sus parámetros
   → Identificar el esquema de autenticación (JWT, API key, OAuth...)

2. Autenticación
   → Registrar al menos 2 cuentas de distinto nivel
   → Obtener tokens para ambas
   → Probar JWT attacks si se usa JWT

3. Autorización (el más importante en APIs)
   → Probar BOLA: acceder a objetos de la segunda cuenta con la primera
   → Probar BFLA: acceder a endpoints de admin con cuenta normal
   → Configurar Autorize en Burp con el token de baja privilegio

4. Validación de input
   → Mass assignment: añadir campos extra en PUT/POST
   → Fuzzing de parámetros con Arjun
   → SQLi, Command Injection, SSTI en los campos

5. Rate limiting y lógica
   → Probar si hay rate limiting en login, registro, reset de contraseña
   → Identificar flujos de negocio abusables a escala

6. Configuración
   → Verificar CORS, métodos HTTP innecesarios
   → Buscar versiones antiguas de la API (/v1/, /v0/, /beta/)
   → Comprobar si los errores revelan información interna
```

> API1 (BOLA) es la vulnerabilidad más prevalente en APIs según el OWASP — más común que SQLi en aplicaciones modernas. Es el primer sitio donde buscar en cualquier pentest de API.

> Si la app tiene documentación Swagger expuesta, el trabajo de reconocimiento es trivial: ya tienes todos los endpoints, parámetros y esquemas de datos. Buscar siempre `/swagger-ui.html` y `/api-docs` antes de hacer fuzzing.

> La principal diferencia táctica entre atacar una web y una API es la automatización: un atacante puede probar 10.000 IDs en segundos contra una API sin defensa. Siempre verificar si hay rate limiting antes de hacer fuzzing masivo.
