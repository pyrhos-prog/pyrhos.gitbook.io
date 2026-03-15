---
icon: spider-web
---

# Reconocimiento

## Metodología Completa de Enumeración Web

### Fases de la Enumeración

```
1. PASIVO (No alertas)
   └─ Subfinder → DNSx

2. VALIDACIÓN HTTP (Bajo-medio ruido)
   └─ httpx → Filtrado

3. ANÁLISIS VISUAL (Bajo-medio ruido)
   └─ GoWitness → Screenshots + DB

4. DESCOBRIMIENTO ADICIONAL (Medio ruido)
   └─ ParamSpider → Parámetros
   └─ Ffuf → Directorios/Vhosts
   └─ Nuclei → Vulnerabilidades

5. ANÁLISIS TÉCNICO
   └─ Waf-detect → Identificar WAF
   └─ Retire.js → JS vulnerables
   └─ Análisis manual en Burp
```

### Requisitos previos

```bash
# Instalación de herramientas
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/sensepost/gowitness@latest
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install -v github.com/projectdiscovery/paramspider@latest
go install -v github.com/ffuf/ffuf@latest
go install github.com/projectdiscovery/wafw00f/cmd/wafw00f@latest

# En Fedora/RHEL
sudo dnf install -y openjdk-latest-runtime  # Para algunas herramientas

# Opcional pero recomendado
git clone https://github.com/RetireJS/retire.js.git
```

### FASE 1: RECONOCIMIENTO PASIVO

#### Paso 1.1: Enumeración de subdominios con Subfinder

```bash
TARGET="target.com"
WORKSPACE="$TARGET-enum-$(date +%Y%m%d)"

mkdir -p "$WORKSPACE"
cd "$WORKSPACE"

echo "[+] Fase 1: Reconocimiento pasivo con Subfinder"
echo "[*] Target: $TARGET"

subfinder -d $TARGET -silent -o 01-subdominios-raw.txt

echo "[✓] Subdominios encontrados: $(wc -l < 01-subdominios-raw.txt)"
```

**Salida esperada:**

* `01-subdominios-raw.txt` - Lista de subdominios sin validar

***

#### Paso 1.2: Resolver DNS y validar existencia

```bash
# Resolver A records de todos los subdominios
dnsx -l 01-subdominios-raw.txt -a -o 01-dns-resolved.txt

# Resolver MX records (opcional, util para email enumeration)
dnsx -l 01-subdominios-raw.txt -mx -o mx-records.txt

# Ver cuáles resuelven
echo "[✓] Subdominios resueltos: $(wc -l < 01-dns-resolved.txt)"

# Filtrar solo los que resuelven
cat 01-dns-resolved.txt | cut -d' ' -f1 | sort -u > 01-subdominios-validos.txt
```

**Archivos generados:**

* `01-dns-resolved.txt` - IPs resueltas
* `01-subdominios-validos.txt` - Solo subdominios que resuelven

### FASE 2: VALIDACIÓN HTTP

#### Paso 2.1: httpx - Verificar status HTTP

```bash
echo "[+] Fase 2: Validación HTTP con httpx"

httpx -l 01-subdominios-validos.txt \
  -fc 404,403,500,502,503,504 \
  -status-code \
  -title \
  -content-length \
  -tech-detect \
  -location \
  -rate-limit 50 \
  -timeout 5 \
  -threads 25 \
  -retries 2 \
  -json \
  -o 02-httpx-results.json

echo "[✓] URLs vivas encontradas"
```

**Salida: `02-httpx-results.json`**

```json
{
  "url": "https://api.target.com",
  "status_code": 200,
  "title": "API Dashboard",
  "content_length": 45123,
  "technologies": ["Node.js", "Express"],
  "location": "",
  "time": "45ms"
}
```

***

#### Paso 2.2: Parsear y filtrar resultados

```bash
# Extraer solo URLs vivas (200, 301, 302, 401, 403 interesantes)
jq -r 'select(.status_code | IN(200,301,302,401,403,429)) | .url' \
  02-httpx-results.json | sort -u > 02-urls-vivas.txt

echo "[✓] URLs vivas: $(wc -l < 02-urls-vivas.txt)"

# Crear index para análisis posterior
jq -r '.url + " [" + (.status_code | tostring) + "] " + .title' \
  02-httpx-results.json > 02-urls-indexed.txt

cat 02-urls-indexed.txt
```

**Archivo: `02-urls-vivas.txt`**

```
https://target.com
https://api.target.com
https://admin.target.com
https://staging.target.com
```

### FASE 3: ANÁLISIS VISUAL CON GOWITNESS

#### Paso 3.1: Tomar screenshots

```bash
echo "[+] Fase 3: Screenshots con GoWitness v3"

gowitness scan file -f 02-urls-vivas.txt \
  --threads 8 \
  --delay 500 \
  --timeout 20 \
  --writer-db \
  --writer-jsonl \
  --destination ./screenshots

echo "[✓] Screenshots completados"
```

**Genera:**

* `gowitness.sqlite` - Base de datos con metadata
* `screenshots/` - Directorio con PNGs
* `.jsonl` - Línea por línea en JSON

#### Paso 3.2: Análisis en interfaz web

```bash
# Iniciar servidor de reportes
echo "[+] Iniciando GoWitness report server..."
gowitness report --port 7171

# Abrir en navegador
# http://localhost:7171

# En otra terminal, mantener acceso:
# firefox http://localhost:7171 &
```

**En la UI de GoWitness:**

1. **Dashboard** - Estadísticas generales
2. **Table View** - Listar todos con filtros
3. **Detail** - Inspeccionar individual
4. **Search** - Buscar por headers, tecnologías, etc
5. **Export** - Exportar para análisis

#### Paso 3.3: Queries SQL en GoWitness

```bash
# Acceder a la DB directamente
sqlite3 gowitness.sqlite

# Queries útiles:
SELECT url, status_code, title, technologies 
FROM urls 
WHERE status_code = 200 
ORDER BY url;

# URLs con keywords interesantes
SELECT url, title 
FROM urls 
WHERE title LIKE '%admin%' OR title LIKE '%login%';

# Tecnologías detectadas
SELECT DISTINCT technologies 
FROM urls 
WHERE technologies IS NOT NULL;

# Exportar a CSV
.mode csv
.output report.csv
SELECT url, status_code, title, technologies FROM urls;
.quit
```

### FASE 4: DESCUBRIMIENTO ADICIONAL

#### Paso 4.1: WAF Detection

```bash
echo "[+] Fase 4.1: Detectando WAF..."

wafw00f -l 02-urls-vivas.txt -o 04-waf-detect.txt

cat 04-waf-detect.txt
```

**Importante:** Si hay WAF, ajustar estrategia (rate limiting, User-Agent, etc)

#### Paso 4.2: Descubrimiento de parámetros (ParamSpider)

```bash
echo "[+] Fase 4.2: Extrayendo parámetros con ParamSpider..."

# ParamSpider usa Wayback Machine - pasivo
paramspider --domain target.com --output params.txt --quiet

# Filtrar parámetros únicos
sort -u params.txt > 04-parametros-unicos.txt

echo "[✓] Parámetros encontrados: $(wc -l < 04-parametros-unicos.txt)"
```

**Archivo: `04-parametros-unicos.txt`**

```
user_id
product_id
redirect_url
search
q
sort_by
filter
```

#### Paso 4.3: Fuzzing de directorios y vhosts

```bash
echo "[+] Fase 4.3: Fuzzing con Ffuf..."

# Crear wordlist para directorios
# Usar: /usr/share/seclists/Discovery/Web-Content/common.txt
# O descarga: https://github.com/danielmiessler/SecLists

# Fuzzing de directorios (en cada URL viva)
for url in $(cat 02-urls-vivas.txt); do
  echo "[*] Fuzzing: $url"
  ffuf -u "$url/FUZZ" \
    -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -fc 404 \
    -t 50 \
    -of json \
    -o "04-ffuf-$(echo $url | md5sum | cut -d' ' -f1).json"
done

# Consolidar resultados
echo "[✓] Fuzzing completado"
```

**Filtrar resultados interesantes:**

```bash
# Directorios con status 200 o 301
find . -name "04-ffuf-*.json" -exec jq '.results[] | select(.status | IN(200,301))' {} \; \
  > 04-directorios-interesantes.txt
```

#### Paso 4.4: Fuzzing de Vhosts

```bash
echo "[+] Fase 4.4: Fuzzing de Virtual Hosts..."

# Obtener IP principal de target.com
IP=$(dig +short target.com A | head -1)

# Fuzzing de vhosts
ffuf -u "http://$IP/" \
  -H "Host: FUZZ.target.com" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fc 404 \
  -t 50 \
  -of json \
  -o 04-vhosts-ffuf.json
```

### FASE 5: ANÁLISIS TÉCNICO

#### Paso 5.1: Nuclei - Escaneo de vulnerabilidades

```bash
echo "[+] Fase 5.1: Escaneo con Nuclei..."

# Actualizar templates
nuclei -update-templates

# Escaneo básico
nuclei -l 02-urls-vivas.txt \
  -t /root/nuclei-templates/cves/ \
  -t /root/nuclei-templates/exposures/ \
  -o 05-nuclei-results.txt \
  -severity high,critical

# Escaneo específico (CORS, CSP, etc)
nuclei -l 02-urls-vivas.txt \
  -t /root/nuclei-templates/http/ \
  -tags cors,csp,headers \
  -o 05-nuclei-headers.txt
```

**Ejemplo de output:**

```
[CVE-2021-1234] https://admin.target.com - Vulnerable to XXE
[CORS-Misconfiguration] https://api.target.com - Overly permissive CORS
[Missing-Security-Headers] https://target.com - Missing CSP header
```

#### Paso 5.2: Análisis de JavaScript (Retire.js)

```bash
echo "[+] Fase 5.2: Escaneo de librerías JS vulnerables..."

# Descargar retire.js si no está
[ ! -d retire.js ] && git clone https://github.com/RetireJS/retire.js.git

cd retire.js
npm install

# Escanear las URLs (descarga JS y analiza)
node retire.js --jspath screenshots/ --outputformat json > ../05-retire-results.json

cd ..
```

#### Paso 5.3: Extracción de tecnologías para análisis adicional

```bash
echo "[+] Fase 5.3: Resumen de tecnologías..."

# Desde httpx results
jq -r '.technologies[]?' 02-httpx-results.json | sort | uniq -c | sort -rn > 05-tech-summary.txt

cat 05-tech-summary.txt
```

**Ejemplo:**

```
10 Apache
8  PHP 7.4
7  jQuery 3.5.1
5  Bootstrap 4
3  Wordpress 5.8
```

***

### FASE 6: COMPILACIÓN Y ANÁLISIS

#### Paso 6.1: Crear reporte consolidado

```bash
cat > 06-reporte-enumeracion.txt << 'EOF'
========================================
REPORTE DE ENUMERACIÓN
========================================

TARGET: target.com
FECHA: $(date)
ENUMERADOR: Pyrhos

--- ESTADÍSTICAS ---
Total subdominios encontrados: $(wc -l < 01-subdominios-raw.txt)
Subdominios que resuelven: $(wc -l < 01-subdominios-validos.txt)
URLs HTTP vivas: $(wc -l < 02-urls-vivas.txt)
Directorios descubiertos: $(jq '.results[] | select(.status == 200)' 04-ffuf-*.json | wc -l)
Vulnerabilidades encontradas: $(wc -l < 05-nuclei-results.txt)

--- URLS VIVAS PRIORITARIAS ---
$(head -20 02-urls-indexed.txt)

--- TECNOLOGÍAS DETECTADAS ---
$(head -10 05-tech-summary.txt)

--- WAF DETECTADO ---
$(cat 04-waf-detect.txt | head -5)

--- VULNERABILIDADES CRÍTICAS ---
$(grep -E "critical|high" 05-nuclei-results.txt | head -10)

========================================
EOF

cat 06-reporte-enumeracion.txt
```

#### Paso 6.2: Estructura final de directorios

```
target.com-enum-20260315/
├── 01-subdominios-raw.txt
├── 01-dns-resolved.txt
├── 01-subdominios-validos.txt
├── 02-httpx-results.json
├── 02-urls-vivas.txt
├── 02-urls-indexed.txt
├── 03-screenshots/
│   ├── https-target.com.png
│   ├── https-api.target.com.png
│   └── ...
├── gowitness.sqlite
├── 04-waf-detect.txt
├── 04-parametros-unicos.txt
├── 04-directorios-interesantes.txt
├── 04-vhosts-ffuf.json
├── 05-nuclei-results.txt
├── 05-nuclei-headers.txt
├── 05-retire-results.json
├── 05-tech-summary.txt
├── 06-reporte-enumeracion.txt
└── notes.txt (anotaciones manuales)
```

### HERRAMIENTAS COMPLEMENTARIAS

Una vez completada la enumeración, para profundizar:

#### Burp Suite Pro

* Análisis manual detallado
* Intruder para fuzzing
* Scanner para vulnerabilidades

#### Análisis de Respuestas

```bash
# Extraer todos los headers interesantes
jq -r '.headers | to_entries[] | "\(.key): \(.value)"' 02-httpx-results.json | sort -u

# Headers de seguridad
jq -r 'select(.headers | keys[] | contains("X-") or contains("Content-Security-Policy")) | .url + ": " + (.headers | to_entries[].key)' 02-httpx-results.json
```

#### Testing Manual

* Validar hallazgos de Nuclei
* Buscar lógica de negocio
* Testing de autenticación/autorización

### TIPS FINALES

**Documentación:**

* Guardar notas de cada descubrimiento
* Screenshots manuales de paneles interesantes
* Guardar URLs de endpoints críticos

**Ruido & Detección:**

* Este pipeline genera entre 2000-3000 requests
* Duración típica: 30-60 minutos
* Visible en logs pero pasa como tráfico legítimo
