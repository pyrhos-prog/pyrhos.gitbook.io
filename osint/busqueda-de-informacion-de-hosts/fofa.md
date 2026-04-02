---
icon: magnifying-glass
---

# FOFA

> **FOFA** (_Fingerprint Of All_) es un motor de búsqueda de ciberespacio desarrollado en China por Baimaohui Security Research Center. Indexa más de **10.000 millones de activos** de internet, con cobertura especialmente fuerte en infraestructura asiática y en protocolos industriales. Es más accesible que Shodan en cuentas gratuitas y ofrece un sistema de fingerprinting de aplicaciones (`app=`) muy potente.

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

### Diferencias clave respecto a Shodan

| Característica                  | FOFA                  | Shodan                 |
| ------------------------------- | --------------------- | ---------------------- |
| Cobertura Asia/China            | ✅ Superior            | ⚠️ Parcial             |
| Fingerprinting de apps (`app=`) | ✅ Nativo              | ❌ No existe            |
| Filtros gratuitos               | ✅ Más permisivos      | ⚠️ Limitados           |
| Cobertura global general        | ⚠️ Buena              | ✅ Superior             |
| Protocolo ICS/OT gratuito       | ✅ Sí                  | ❌ Solo enterprise      |
| Hash de favicon                 | ✅ `icon_hash=`        | ✅ `http.favicon.hash:` |
| JARM fingerprint                | ✅ `jarm=`             | ✅ `ssl.jarm:`          |
| Honeypot detection              | ✅ `is_honeypot=false` | ⚠️ `tag:honeypot`      |

### Acceso

| Método           | URL                                                              |
| ---------------- | ---------------------------------------------------------------- |
| Web UI (inglés)  | [en.fofa.info](https://en.fofa.info/)                            |
| Web UI (chino)   | [fofa.info](https://fofa.info/)                                  |
| API REST         | `https://fofa.info/api/v1/search/all`                            |
| CLI oficial (Go) | [github.com/FofaInfo/GoFOFA](https://github.com/FofaInfo/GoFOFA) |
| CLI comunitario  | [fofax](https://github.com/xiecat/fofax)                         |

### Sintaxis de búsqueda

FOFA usa una sintaxis propia basada en **pares `campo="valor"`**. A diferencia de Shodan (que usa `campo:valor`), FOFA usa `=` con comillas.

```
campo="valor"
campo="valor1" && campo2="valor2"     # AND
campo="valor1" || campo2="valor2"     # OR
campo="valor1" && campo2!="valor2"    # NOT en valor específico
```

**Wildcards (fuzzy search):**

```
domain="*.empresa.com"      # wildcard al inicio
domain="empresa.*"          # wildcard al final
version="5.?.*"             # ? = un carácter, * = varios
```

> En FOFA, `*` y `?` se usan dentro del valor de la búsqueda, no como prefijo del campo.

### Operadores y comparadores

| Operador  | Significado                               | Ejemplo                         |
| --------- | ----------------------------------------- | ------------------------------- |
| `=`       | Igual (exact match o fuzzy con wildcards) | `title="Login"`                 |
| `!=`      | Distinto de                               | `port!="443"`                   |
| `&&`      | AND lógico                                | `title="nginx" && country="ES"` |
| `\|\|`    | OR lógico                                 | `port="80" \|\| port="8080"`    |
| `=*`      | Fuzzy/wildcard                            | `domain="*.gov.es"`             |
| `after=`  | Resultados después de fecha               | `after="2024-01-01"`            |
| `before=` | Resultados antes de fecha                 | `before="2024-06-01"`           |

### Filtros generales

Aplicables a cualquier tipo de búsqueda.

| Filtro        | Descripción                                   | Ejemplo                                |
| ------------- | --------------------------------------------- | -------------------------------------- |
| `ip`          | Dirección IP exacta o rango CIDR              | `ip="1.2.3.4"` / `ip="192.168.0.0/24"` |
| `port`        | Puerto                                        | `port="8080"`                          |
| `domain`      | Dominio o subdominio                          | `domain="empresa.es"`                  |
| `host`        | Host completo (incluye puerto)                | `host="empresa.es:8443"`               |
| `title`       | Título HTML de la página                      | `title="Dashboard"`                    |
| `body`        | Contenido del cuerpo HTML                     | `body="Powered by"`                    |
| `header`      | Cabeceras HTTP de respuesta                   | `header="X-Powered-By: PHP"`           |
| `banner`      | Banner completo del servicio                  | `banner="SSH-2.0-OpenSSH"`             |
| `protocol`    | Protocolo detectado                           | `protocol="https"`                     |
| `country`     | Código de país ISO                            | `country="ES"`                         |
| `region`      | Región/provincia                              | `region="Madrid"`                      |
| `city`        | Ciudad                                        | `city="Madrid"`                        |
| `os`          | Sistema operativo                             | `os="Windows Server 2019"`             |
| `server`      | Servidor web (del header)                     | `server="Apache"`                      |
| `app`         | Aplicación/producto detectado por fingerprint | `app="WordPress"`                      |
| `product`     | Nombre de producto (alternativa a `app`)      | `product="nginx"`                      |
| `version`     | Versión del producto                          | `version="2.4.49"`                     |
| `icp`         | Número de registro ICP (China)                | `icp="京ICP备"`                          |
| `org`         | Organización/ASN                              | `org="Telefonica"`                     |
| `asn`         | Número AS                                     | `asn="AS3352"`                         |
| `is_honeypot` | Filtrar honeypots                             | `is_honeypot=false`                    |
| `is_domain`   | Solo resultados con dominio resuelto          | `is_domain=true`                       |
| `is_ipv6`     | Solo IPv6                                     | `is_ipv6=true`                         |
| `after`       | Datos actualizados después de fecha           | `after="2024-01-01"`                   |
| `before`      | Datos actualizados antes de fecha             | `before="2024-12-31"`                  |

### Filtros por tipo de búsqueda

#### Servidores web y aplicaciones

```fofa
title="phpMyAdmin" && country="ES"
app="WordPress" && country="ES"
body="Index of /" && title="Apache"
header="X-Powered-By: ThinkPHP"
```

| Filtro        | Descripción                                 | Ejemplo                     |
| ------------- | ------------------------------------------- | --------------------------- |
| `title`       | Título HTML                                 | `title="Login Panel"`       |
| `body`        | HTML del body                               | `body="wp-content"`         |
| `header`      | Cabecera HTTP específica                    | `header="Server: IIS"`      |
| `app`         | Fingerprint de aplicación (Wappalyzer-like) | `app="Drupal"`              |
| `server`      | Header Server                               | `server="nginx"`            |
| `icon_hash`   | Hash MurmurHash3 del favicon                | `icon_hash="-247388890"`    |
| `header_hash` | Hash de las cabeceras HTTP                  | `header_hash="-1524283538"` |
| `body_hash`   | Hash del cuerpo de la respuesta             | `body_hash="1234567890"`    |
| `status_code` | Código de respuesta HTTP                    | `status_code="200"`         |
| `js_name`     | Nombre de librería JS detectada             | `js_name="jquery"`          |
| `js_md5`      | MD5 de un fichero JS específico             | `js_md5="abc123..."`        |

> **Truco — icon\_hash en FOFA:**
>
> ```python
> import mmh3, requests, base64
> r = requests.get("https://target.com/favicon.ico")
> favicon = base64.encodebytes(r.content)
> print(mmh3.hash(favicon))
> ```
>
> Busca ese hash con `icon_hash="-123456789"` para encontrar todos los activos de un target, incluso los que no aparecen en DNS.

#### SSL/TLS y certificados

```fofa
cert="empresa.com" && country="ES"
cert.subject.org="Target Corp" && is_honeypot=false
cert.issuer.cn="Let's Encrypt" && domain="*.empresa.es"
cert.is_valid=false && domain="empresa.com"
```

| Filtro             | Descripción                                      | Ejemplo                           |
| ------------------ | ------------------------------------------------ | --------------------------------- |
| `cert`             | Cualquier campo del certificado (búsqueda libre) | `cert="empresa"`                  |
| `cert.subject.org` | Organización del subject                         | `cert.subject.org="Target"`       |
| `cert.subject.cn`  | Common Name del subject                          | `cert.subject.cn="*.empresa.es"`  |
| `cert.issuer.cn`   | Common Name del emisor                           | `cert.issuer.cn="DigiCert"`       |
| `cert.issuer.org`  | Organización del emisor                          | `cert.issuer.org="Let's Encrypt"` |
| `cert.is_valid`    | Certificado válido/caducado                      | `cert.is_valid=true`              |
| `cert.is_expired`  | Certificado caducado explícitamente              | `cert.is_expired=true`            |
| `cert.is_match`    | Cert coincide con el dominio                     | `cert.is_match=false`             |
| `jarm`             | Hash JARM del servidor TLS                       | `jarm="29d29d00029d..."`          |
| `tls.version`      | Versión TLS/SSL                                  | `tls.version="TLSv1"`             |

> **Enumeración de activos con wildcards:** `cert.subject.cn="*.empresa.com"` encuentra todos los activos de una empresa que usen ese certificado wildcard, incluyendo subdominos no enumerados por DNS.

#### SSH

```fofa
protocol="ssh" && country="ES" && banner="SSH-2.0-OpenSSH_7"
app="OpenSSH" && version="7.4" && country="ES"
```

| Filtro     | Descripción                  | Ejemplo                    |
| ---------- | ---------------------------- | -------------------------- |
| `protocol` | Protocolo SSH                | `protocol="ssh"`           |
| `port`     | Puerto SSH                   | `port="22"`                |
| `banner`   | Banner completo del servicio | `banner="SSH-2.0-OpenSSH"` |
| `app`      | Implementación SSH detectada | `app="OpenSSH"`            |
| `version`  | Versión de OpenSSH           | `version="8.9p1"`          |

#### FTP

```fofa
protocol="ftp" && banner="220" && country="ES"
banner="230 Login successful" && banner="anonymous"
```

| Filtro     | Descripción             | Ejemplo                |
| ---------- | ----------------------- | ---------------------- |
| `protocol` | Protocolo FTP           | `protocol="ftp"`       |
| `port`     | Puerto FTP              | `port="21"`            |
| `banner`   | Banner de respuesta FTP | `banner="220 ProFTPD"` |

#### Bases de datos

**MongoDB**

```fofa
app="MongoDB" && country="ES"
port="27017" && app="MongoDB"
```

**Elasticsearch / Kibana**

```fofa
app="Elastic" && port="9200" && country="ES"
title="Kibana" && port="5601"
app="Kibana" && country="ES"
```

**Redis**

```fofa
app="Redis" && country="ES"
port="6379" && banner="redis_version"
```

**MySQL / MSSQL / PostgreSQL**

```fofa
app="MySQL" && country="ES"
app="Microsoft-SQL-Server" && country="ES"
app="PostgreSQL" && country="ES"
```

**Otras**

```fofa
app="Apache-CouchDB" && country="ES"
app="Memcached" && country="ES"
app="InfluxDB" && country="ES"
```

| Base de datos | `app=` value             | Puerto por defecto |
| ------------- | ------------------------ | ------------------ |
| MongoDB       | `"MongoDB"`              | 27017              |
| Elasticsearch | `"Elastic"`              | 9200               |
| Kibana        | `"Kibana"`               | 5601               |
| Redis         | `"Redis"`                | 6379               |
| MySQL         | `"MySQL"`                | 3306               |
| MSSQL         | `"Microsoft-SQL-Server"` | 1433               |
| PostgreSQL    | `"PostgreSQL"`           | 5432               |
| Memcached     | `"Memcached"`            | 11211              |
| CouchDB       | `"Apache-CouchDB"`       | 5984               |
| Cassandra     | `"Cassandra"`            | 9042               |

#### Escritorio remoto y gestión

```fofa
app="Remote-Desktop-Protocol" && country="ES"
app="VNC" && country="ES"
protocol="rdp" && country="ES"
```

| Servicio   | Filtro                                   | Puerto    |
| ---------- | ---------------------------------------- | --------- |
| RDP        | `app="Remote-Desktop-Protocol"`          | 3389      |
| VNC        | `app="VNC"`                              | 5900      |
| Telnet     | `protocol="telnet"`                      | 23        |
| WinRM      | `port="5985" \|\| port="5986"`           | 5985/5986 |
| IPMI       | `port="623"`                             | 623       |
| TeamViewer | `app="TeamViewer"`                       | 5938      |
| RDP (alt)  | `title="Remote Desktop" && country="ES"` | —         |

#### Paneles de administración y virtualización

```fofa
app="Proxmox-Virtual-Environment" && country="ES"
app="VMware-ESXi" && country="ES"
title="Grafana" && country="ES"
app="phpMyAdmin" && country="ES"
title="Jupyter Notebook" && country="ES"
```

| Panel          | Filtro app/title                    |
| -------------- | ----------------------------------- |
| Proxmox        | `app="Proxmox-Virtual-Environment"` |
| VMware ESXi    | `app="VMware-ESXi"`                 |
| vCenter        | `app="VMware-vCenter"`              |
| Grafana        | `app="Grafana"`                     |
| Kibana         | `app="Kibana"`                      |
| phpMyAdmin     | `app="phpMyAdmin"`                  |
| Jupyter        | `title="Jupyter Notebook"`          |
| Jenkins        | `app="Jenkins"`                     |
| GitLab         | `app="GitLab"`                      |
| Docker API     | `port="2375" \|\| port="2376"`      |
| Kubernetes API | `port="6443" && app="Kubernetes"`   |
| Portainer      | `app="Portainer"`                   |
| pfSense        | `title="pfSense"`                   |

#### ICS/SCADA y sistemas industriales

> Solo para uso en assets propios o con autorización explícita. Infraestructura crítica.

```fofa
app="Industrial-Control-Products" && country="ES"
protocol="modbus" && country="ES"
protocol="s7" && country="DE"
app="Siemens-S7-PLC"
```

| Protocolo   | Filtro                                   | Puerto |
| ----------- | ---------------------------------------- | ------ |
| Modbus      | `protocol="modbus"`                      | 502    |
| Siemens S7  | `protocol="s7"` / `app="Siemens-S7-PLC"` | 102    |
| DNP3        | `protocol="dnp3"`                        | 20000  |
| BACnet      | `protocol="bacnet"`                      | 47808  |
| EtherNet/IP | `protocol="enip"`                        | 44818  |
| IEC 104     | `protocol="iec-104"`                     | 2404   |
| HART-IP     | `protocol="hart-ip"`                     | 5094   |
| Todos ICS   | `app="Industrial-Control-Products"`      | —      |

> **Ventaja FOFA vs Shodan:** el filtro `app="Industrial-Control-Products"` es accesible en plan gratuito, mientras que en Shodan el tag `ics` es solo enterprise.

#### Threat Hunting — C2 e infraestructura maliciosa

```fofa
# Cobalt Strike (JARM fingerprint)
jarm="07d14d16d21d21d07c42d41d00041d24a458a375eef0c576d23a7bab9a9fb1"

# Cobalt Strike (cert)
cert="Major Cobalt Strike"

# Cobalt Strike (título)
title="Cobalt Strike"

# Brute Ratel C4
jarm="2ad2ad0002ad2ad00042d42d000000ad9bf51cc3f5a1e29e4afa3e701e8a82"

# Gophish (phishing framework)
title="gophish" || body="gophish"

# Evilginx2
cert="evilginx" || body="X-Evilginx"

# Web shells comunes
body="c99" || body="r57" || body="WSO"
body="eval(base64_decode" && body="$_POST"

# Paneles de phishing
title="Login" && body="Microsoft" && cert.issuer!="Microsoft"
```

| Infraestructura      | Filtro FOFA                      |
| -------------------- | -------------------------------- |
| Cobalt Strike        | `jarm="07d14d16d21d21d..."`      |
| Cobalt Strike (cert) | `cert="Major Cobalt Strike"`     |
| Brute Ratel          | `jarm="2ad2ad0002ad..."`         |
| Gophish              | `title="gophish"`                |
| Evilginx             | `cert="evilginx"`                |
| Havoc C2             | `header="X-Havoc"`               |
| Sliver C2            | `cert.subject.org="multiplayer"` |

### El sistema `app=` — fingerprints de FOFA

`app=` es la característica más potente y diferenciadora de FOFA. Usa una base de datos interna de fingerprints para identificar aplicaciones por sus características (títulos, headers, body, certificados, iconos...).

```fofa
app="WordPress" && country="ES"
app="Apache-Struts" && country="ES"       # CVE-2017-5638
app="Log4j" && country="ES"               # CVE-2021-44228
app="Spring-Boot" && country="ES"
app="Citrix-NetScaler-Gateway"            # CVE-2023-3519
app="Fortinet-FortiGate"
app="Pulse-Secure-VPN"
app="F5-BIG-IP"
```

> La lista completa de fingerprints está en [en.fofa.info/library](https://en.fofa.info/library). Actualmente supera los 10.000 fingerprints.

### API REST

```bash
BASE: https://fofa.info/api/v1/search/all

# Parámetros:
# email=    Tu email de cuenta FOFA
# key=      Tu API key
# qbase64=  Query en base64
# size=     Número de resultados (max 10000 por query con F-points)
# page=     Página de resultados
# fields=   Campos a devolver (ip,port,host,title,domain,...)
# full=     true para incluir datos históricos

# Ejemplo cURL
curl "https://fofa.info/api/v1/search/all?email=TU_EMAIL&key=TU_KEY&qbase64=$(echo -n 'app="nginx" && country="ES"' | base64)&size=100&fields=ip,port,host,title"
```

#### Campos disponibles en la API

```
ip, port, protocol, country, country_name, region, city,
longitude, latitude, as_number, as_organization,
host, domain, os, server, icp, title, jarm, header, banner,
cert, base_protocol, link, product, product_category,
version, lastupdatetime, cname, icon_hash, certs_valid,
cname_domain, body, fid, structinfo
```

> Queries con `cert` o `banner` → máx. 2000 resultados/página. Queries con `body` → máx. 500 resultados/página.

### GoFOFA — CLI oficial

```bash
# Instalar
go install github.com/FofaInfo/GoFOFA/cmd/fofa@latest

# Configurar API key
export FOFA_KEY="TU_API_KEY"
export FOFA_EMAIL="TU_EMAIL"

# Búsqueda básica (100 resultados, deduplicados por defecto)
fofa search -s 100 'app="nginx" && country="ES"'

# Campos personalizados
fofa search -f ip,port,host,title,protocol 'title="Login" && country="ES"'

# Exportar a CSV
fofa dump -f ip,port,host,title -o resultado.csv 'app="WordPress" && country="ES"'

# Exportar a JSON
fofa dump -f ip,port,host -o resultado.json --format json 'port="22" && country="ES"'

# Contar resultados sin consumir cuota
fofa count 'app="Apache"'

# Info de un host
fofa host demo.empresa.com

# Calcular icon_hash desde URL
fofa icon --open https://target.com/favicon.ico

# Buscar por icon_hash desde fichero local
fofa search -if ./favicon.ico

# Buscar por certificado de una URL
fofa search -uc https://target.com

# Verificar si hosts están activos (3 reintentos)
fofa --checkActive 3 -s 100 --format=json 'port="80" && country="ES"'

# Datos históricos (incluye hosts offline)
fofa search --full true -s 100 'app="Joomla"'

# Estadísticas (facets)
fofa stats --field country 'app="Apache-Struts"'
```

### fofax — CLI comunitario popular

```bash
# Instalar
go install github.com/xiecat/fofax/cmd/fofax@latest

# Búsqueda básica
fofax -q 'app="APACHE-Solr"'

# Excluir honeypots
fofax -q 'app="Redis"' -e

# Excluir China
fofax -q 'app="Redis"' -ec

# Obtener solo IPs
fofax -q 'port="3389" && country="ES"' -ffi

# Obtener títulos de los dominios
fofax -q 'domain="empresa.com"' -fto

# Fetch JARM de los resultados
fofax -q 'app="Nginx" && country="ES"' -fjo

# Campos custom
fofax -q 'app="Spring-Boot"' -ff "host,lastupdatetime"
```

### FOFA Dorks útiles

```fofa
# Jenkins sin login
app="Jenkins" && title="Dashboard" && body!="login"

# GitLab expuesto
app="GitLab" && title!="Sign in"

# Paneles Grafana
app="Grafana" && country="ES"

# Jupyter Notebook sin auth
title="Jupyter Notebook" && body!="password"

# Docker API expuesta
port="2375" && protocol="http"

# Kubernetes API sin TLS
port="8080" && app="Kubernetes"

# Spring Boot Actuator expuesto
body="/actuator" && app="Spring-Boot"

# phpMyAdmin
app="phpMyAdmin" && country="ES"

# Hadoop NameNode
app="Hadoop" && port="50070"

# Elasticsearch sin auth
app="Elastic" && port="9200" && country="ES"

# Log4Shell — versiones afectadas
app="Log4j" && version="2.1*"

# Apache Struts (CVE-2017-5638)
app="Apache-Struts" && country="ES"

# VMware vCenter (CVE-2021-21985)
app="VMware-vCenter"

# Fortinet con credenciales expuestas
app="Fortinet-FortiGate" && body="credentials"

# SMTP open relay
protocol="smtp" && banner="220" && banner="ESMTP" && country="ES"

# Paneles con "admin/admin" en el body (evidencia de config por defecto)
body="admin" && title="admin" && app="Router"
```

### Comparativa de sintaxis FOFA vs Shodan

| Búsqueda          | FOFA                  | Shodan                                     |
| ----------------- | --------------------- | ------------------------------------------ |
| Título web        | `title="Login"`       | `http.title:"Login"`                       |
| Cuerpo HTML       | `body="content"`      | `http.html:"content"`                      |
| Certificado       | `cert="empresa"`      | `ssl:"empresa"`                            |
| País              | `country="ES"`        | `country:ES`                               |
| Puerto            | `port="22"`           | `port:22`                                  |
| Sistema operativo | `os="Windows"`        | `os:"Windows"`                             |
| Aplicación        | `app="WordPress"`     | ❌ (no existe)                              |
| Favicon hash      | `icon_hash="-12345"`  | `http.favicon.hash:-12345`                 |
| JARM              | `jarm="07d14..."`     | `ssl.jarm:07d14...`                        |
| Rango CIDR        | `ip="192.168.0.0/24"` | `net:192.168.0.0/24`                       |
| Organización      | `org="Telefonica"`    | `org:"Telefonica"`                         |
| AND               | `&&`                  | espacio                                    |
| OR                | `\|\|`                | `OR`                                       |
| NOT               | `!=`                  | `-`                                        |
| Honeypot excluir  | `is_honeypot=false`   | `tag:honeypot` (negativo: `-tag:honeypot`) |

### Límites por plan

| Plan                | Resultados/día  | Filtros avanzados | `app=` | Honeypot filter | API             |
| ------------------- | --------------- | ----------------- | ------ | --------------- | --------------- |
| Free (registro)     | 1 página (\~10) | ⚠️ Limitados      | ✅      | ❌               | ❌               |
| Basic (\~$9.9/mes)  | Ilimitados      | ✅                 | ✅      | ✅               | ✅ hasta 10K/mes |
| Professional        | Ilimitados      | ✅                 | ✅      | ✅               | ✅ hasta 1M/mes  |
| Business/Enterprise | Ilimitados      | ✅                 | ✅      | ✅               | Sin límite      |

> Los créditos **F-Points** se consumen al usar la API para descargas masivas. Las búsquedas en la web no consumen F-Points.

### Herramientas de terceros para FOFA

| Herramienta                                           | Lenguaje | Descripción                       |
| ----------------------------------------------------- | -------- | --------------------------------- |
| [GoFOFA](https://github.com/FofaInfo/GoFOFA)          | Go       | CLI oficial de FofaInfo           |
| [fofax](https://github.com/xiecat/fofax)              | Go       | CLI comunitario con filtros extra |
| [FofaMap](https://github.com/Hou5e/FofaMap)           | Python   | CLI con integración Nuclei y MCP  |
| [fofa\_viewer](https://github.com/wgpsec/fofa_viewer) | Java     | GUI de escritorio                 |
| [PyFOFA](https://github.com/fofapro/fofa-py)          | Python   | Wrapper Python para la API        |

### Referencias

* [FOFA Web UI](https://en.fofa.info/)
* [FOFA API Docs](https://en.fofa.info/api)
* [FOFA Library (fingerprints)](https://en.fofa.info/library)
* [Awesome-FOFA (repositorio oficial)](https://github.com/FofaInfo/Awesome-FOFA)
* [GoFOFA CLI](https://github.com/FofaInfo/GoFOFA)
* [fofax CLI](https://github.com/xiecat/fofax)
* Alternativas: [Shodan](https://www.shodan.io/), [Censys](https://censys.io/), [ZoomEye](https://www.zoomeye.org/), [Quake](https://quake.360.net/)
