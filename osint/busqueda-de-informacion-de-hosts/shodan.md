---
icon: magnifying-glass
---

# Shodan

> **Shodan** es un motor de búsqueda para dispositivos conectados a internet. A diferencia de Google, que indexa contenido web, Shodan indexa **banners de servicios**: respuestas de servidores, dispositivos IoT, cámaras, routers, sistemas SCADA, y cualquier cosa expuesta en la red.

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

### ¿Cómo funciona?

Shodan escanea continuamente el espacio de direcciones IPv4 enviando probes a puertos comunes y recoge los **banners de respuesta** (HTTP headers, SSH banners, FTP banners, etc.). Cada resultado incluye:

* Dirección IP y hostname (si resuelve)
* Puertos abiertos y servicios detectados
* País, ciudad, organización/ASN
* Certificados TLS/SSL
* Timestamps del último escaneo
* Vulnerabilidades conocidas (CVEs asociados)

### Acceso

| Método   | URL                                                    |
| -------- | ------------------------------------------------------ |
| Web UI   | [shodan.io](https://www.shodan.io/)                    |
| CLI      | `pip install shodan` → `shodan init <API_KEY>`         |
| API REST | `https://api.shodan.io/shodan/host/{ip}?key=<API_KEY>` |

```bash
# Instalar CLI
pip install shodan

# Inicializar con tu API key
shodan init TU_API_KEY

# Búsqueda básica
shodan search "apache"

# Info de una IP
shodan host 8.8.8.8

# Descargar resultados (requiere créditos)
shodan download resultados.json.gz "nginx country:ES"
shodan parse resultados.json.gz --fields ip_str,port,org
```

### Sintaxis de búsqueda

Las búsquedas en Shodan combinan **términos de texto libre** con **filtros** usando la sintaxis `filtro:valor`. Los filtros se pueden encadenar con espacios (AND implícito).

```
término filtro1:valor1 filtro2:valor2
```

**Ejemplos:**

```
apache port:8080 country:ES
"default password" product:router org:"Telefonica"
```

### Filtros generales

Aplicables a cualquier tipo de búsqueda.

| Filtro           | Descripción                        | Ejemplo                    |
| ---------------- | ---------------------------------- | -------------------------- |
| `country`        | Código de país ISO 3166-1 alpha-2  | `country:ES`               |
| `city`           | Ciudad                             | `city:Madrid`              |
| `geo`            | Coordenadas + radio en km          | `geo:40.41,3.70,50`        |
| `org`            | Nombre de organización (ASN)       | `org:"Telefonica"`         |
| `isp`            | Proveedor de servicios de internet | `isp:"Movistar"`           |
| `net`            | Rango de red en CIDR               | `net:192.168.1.0/24`       |
| `ip`             | Dirección IP específica            | `ip:1.2.3.4`               |
| `hostname`       | Hostname o dominio                 | `hostname:.gov.es`         |
| `asn`            | Número de sistema autónomo         | `asn:AS3352`               |
| `port`           | Puerto específico                  | `port:22`                  |
| `os`             | Sistema operativo detectado        | `os:"Windows Server 2019"` |
| `product`        | Nombre del producto/software       | `product:nginx`            |
| `version`        | Versión del producto               | `version:2.4.49`           |
| `before`         | Resultados antes de fecha          | `before:01/01/2024`        |
| `after`          | Resultados después de fecha        | `after:01/06/2024`         |
| `vuln`           | CVE asociado al host               | `vuln:CVE-2021-44228`      |
| `has_screenshot` | Hosts con captura de pantalla      | `has_screenshot:true`      |
| `tag`            | Etiqueta Shodan interna            | `tag:honeypot`             |

### Filtros por tipo de búsqueda

#### Servidores web

```
http.title:"Dashboard" country:ES
http.status:200 http.favicon.hash:-297xxxxxxxxx
http.html:"Powered by WordPress" country:ES
```

| Filtro              | Descripción                       | Ejemplo                        |
| ------------------- | --------------------------------- | ------------------------------ |
| `http.title`        | Título de la página HTML          | `http.title:"Login"`           |
| `http.html`         | Contenido del cuerpo HTML         | `http.html:"index of /"`       |
| `http.status`       | Código de respuesta HTTP          | `http.status:200`              |
| `http.server`       | Header Server de la respuesta     | `http.server:"Apache/2.4"`     |
| `http.component`    | Tecnología detectada (Wappalyzer) | `http.component:wordpress`     |
| `http.favicon.hash` | Hash MMH3 del favicon             | `http.favicon.hash:116323821`  |
| `http.headers_hash` | Hash de las cabeceras HTTP        | `http.headers_hash:1234567890` |
| `http.waf`          | WAF detectado                     | `http.waf:cloudflare`          |

> Puedes calcular el hash del favicon de un objetivo y buscarlo en Shodan para encontrar todos sus activos expuestos, incluso los que no aparecen en DNS.
>
> ```python
> import mmh3, requests, base64
> r = requests.get("https://target.com/favicon.ico")
> favicon = base64.encodebytes(r.content)
> print(mmh3.hash(favicon))
> ```

#### SSH

```
port:22 product:"OpenSSH" version:"7.4" country:ES
"SSH-2.0-OpenSSH_7.4" country:ES
```

| Filtro    | Descripción                    | Ejemplo             |
| --------- | ------------------------------ | ------------------- |
| `port`    | Puerto SSH (por defecto 22)    | `port:22`           |
| `product` | Implementación SSH             | `product:"OpenSSH"` |
| `version` | Versión de OpenSSH             | `version:"8.9"`     |
| `os`      | Sistema operativo del servidor | `os:"Ubuntu"`       |
| `banner`  | Contenido del banner SSH       | `banner:"SSH-2.0"`  |

#### &#x20;FTP

```
port:21 "230 Login successful" anonymous
"220" "FTP" "anonymous" country:ES
```

| Filtro          | Descripción              | Ejemplo                |
| --------------- | ------------------------ | ---------------------- |
| `port`          | Puerto FTP (21)          | `port:21`              |
| `ftp.anonymous` | Login anónimo habilitado | `ftp.anonymous:true`   |
| `banner`        | Banner del servicio FTP  | `banner:"220 ProFTPD"` |

#### Bases de datos

**MongoDB**

```
port:27017 product:"MongoDB" -authentication
```

| Filtro              | Descripción              | Ejemplo                   |
| ------------------- | ------------------------ | ------------------------- |
| `port`              | Puerto MongoDB           | `port:27017`              |
| `product`           | Producto                 | `product:"MongoDB"`       |
| `mongodb.databases` | Bases de datos expuestas | `mongodb.databases:admin` |

**Elasticsearch**

```
port:9200 product:"Elastic" country:ES
http.title:"Kibana" port:5601
```

| Filtro                  | Descripción                      | Ejemplo                            |
| ----------------------- | -------------------------------- | ---------------------------------- |
| `port`                  | Puerto ES (9200) o Kibana (5601) | `port:9200`                        |
| `product`               | Elastic/Kibana                   | `product:"Kibana"`                 |
| `elasticsearch.cluster` | Nombre del clúster               | `elasticsearch.cluster:production` |

**Redis**

```
port:6379 product:"Redis" -requirepass
```

| Filtro    | Descripción  | Ejemplo           |
| --------- | ------------ | ----------------- |
| `port`    | Puerto Redis | `port:6379`       |
| `product` | Redis        | `product:"Redis"` |

**MySQL / MSSQL / PostgreSQL**

```
port:3306 product:"MySQL"
port:1433 product:"Microsoft SQL Server"
port:5432 product:"PostgreSQL"
```

#### SSL/TLS y certificados

```
ssl.cert.subject.cn:"*.target.com"
ssl.cert.issuer.cn:"Let's Encrypt"
ssl:"target.com" port:443
```

| Filtro                 | Descripción                     | Ejemplo                              |
| ---------------------- | ------------------------------- | ------------------------------------ |
| `ssl`                  | Cualquier campo del certificado | `ssl:"company name"`                 |
| `ssl.cert.subject.cn`  | Common Name del subject         | `ssl.cert.subject.cn:"*.empresa.es"` |
| `ssl.cert.issuer.cn`   | Common Name del emisor          | `ssl.cert.issuer.cn:"DigiCert"`      |
| `ssl.cert.subject.o`   | Organización del subject        | `ssl.cert.subject.o:"Target Corp"`   |
| `ssl.cert.expired`     | Certificado caducado            | `ssl.cert.expired:true`              |
| `ssl.cert.self_signed` | Certificado autofirmado         | `ssl.cert.self_signed:true`          |
| `ssl.jarm`             | Hash JARM del servidor TLS      | `ssl.jarm:29d29d...`                 |
| `ssl.version`          | Versión de TLS/SSL              | `ssl.version:TLSv1`                  |
| `ssl.cipher.name`      | Cipher suite activo             | `ssl.cipher.name:RC4`                |

> **Enumeración de subdominios:** `ssl.cert.subject.cn:"*.empresa.com"` encuentra todos los activos de una empresa que usan certificados wildcard.

#### Protocolos industriales (ICS/SCADA)

> Solo para uso autorizado. Estos sistemas son infraestructura crítica.

```
port:102 product:"Siemens"
port:502 product:"Modbus"
port:44818 product:"EtherNet/IP"
```

| Protocolo       | Puerto | Filtro                     |
| --------------- | ------ | -------------------------- |
| Modbus          | 502    | `port:502 product:Modbus`  |
| DNP3            | 20000  | `port:20000`               |
| S7 (Siemens)    | 102    | `port:102 product:Siemens` |
| BACnet          | 47808  | `port:47808`               |
| EtherNet/IP     | 44818  | `port:44818`               |
| IEC 60870-5-104 | 2404   | `port:2404`                |

***

#### Dispositivos IoT y cámaras

```
"webcam" has_screenshot:true country:ES
http.title:"IP Camera" -password
port:554 product:"RTSP"
```

| Filtro           | Descripción                   | Ejemplo                  |
| ---------------- | ----------------------------- | ------------------------ |
| `has_screenshot` | Tiene captura de pantalla     | `has_screenshot:true`    |
| `product`        | Producto/firmware detectado   | `product:"Hikvision"`    |
| `http.title`     | Título del panel de la cámara | `http.title:"Live View"` |
| `port`           | RTSP (554), ONVIF (80/8080)   | `port:554`               |

#### Escritorio remoto y gestión

```
port:3389 product:"Remote Desktop" country:ES os:"Windows"
port:5900 product:"VNC" -authentication
port:8443 http.title:"pfSense"
```

| Servicio   | Puerto    | Filtro                               |
| ---------- | --------- | ------------------------------------ |
| RDP        | 3389      | `port:3389 product:"Remote Desktop"` |
| VNC        | 5900-5903 | `port:5900 product:"VNC"`            |
| TeamViewer | 5938      | `port:5938 product:"TeamViewer"`     |
| Telnet     | 23        | `port:23`                            |
| WinRM      | 5985/5986 | `port:5985`                          |
| IPMI       | 623       | `port:623`                           |

#### Paneles de administración y gestión cloud

```
http.title:"Proxmox Virtual Environment" country:ES
http.title:"VMware ESXi" -"This page requires"
http.title:"Grafana" country:ES
http.title:"phpMyAdmin"
```

| Panel            | Filtro                                     |
| ---------------- | ------------------------------------------ |
| Proxmox          | `http.title:"Proxmox Virtual Environment"` |
| VMware ESXi      | `http.title:"VMware ESXi"`                 |
| Kibana           | `http.title:"Kibana"` / `port:5601`        |
| Grafana          | `http.title:"Grafana"`                     |
| phpMyAdmin       | `http.title:"phpMyAdmin"`                  |
| Jupyter Notebook | `http.title:"Jupyter"`                     |
| Kubernetes API   | `port:6443 product:"Kubernetes"`           |
| Docker API       | `port:2375`                                |

#### Búsquedas de vulnerabilidades

```
vuln:CVE-2021-44228 country:ES
vuln:CVE-2019-19781
product:"Exchange" version:"15.1" country:ES
```

| Filtro                | Descripción                     | Ejemplo                         |
| --------------------- | ------------------------------- | ------------------------------- |
| `vuln`                | CVE específico asociado al host | `vuln:CVE-2021-26084`           |
| `version`             | Versión vulnerable conocida     | `version:"2.4.49"`              |
| `product` + `version` | Combinación para versiones EOL  | `product:OpenSSL version:1.0.1` |

> El filtro `vuln` requiere **plan Membership** o superior.

### Operadores booleanos y especiales

| Operador | Uso                                | Ejemplo                |
| -------- | ---------------------------------- | ---------------------- |
| `AND`    | Implícito (espacio)                | `apache port:80`       |
| `OR`     | Alternativa entre términos         | `port:80 OR port:8080` |
| `-`      | Exclusión (NOT)                    | `nginx -country:US`    |
| `"..."`  | Frase exacta                       | `"default password"`   |
| `*`      | Wildcard (solo en algunos filtros) | `hostname:*.gov.es`    |

### Shodan CLI — comandos útiles

```bash
# Estadísticas de una búsqueda sin consumir créditos
shodan stats --facets country,port "apache"

# Buscar y guardar
shodan search --limit 100 --fields ip_str,port,org "nginx country:ES" > resultados.txt

# Alertas (monitorización continua)
shodan alert create "Mi red" 192.168.0.0/24
shodan alert list

# Scan bajo demanda (requiere créditos)
shodan scan submit 93.184.216.34

# Información de tu cuenta/créditos
shodan info

# Convertir descarga a CSV
shodan convert resultados.json.gz csv
```

### Shodan Dorks útiles

```bash
# Paneles con credenciales por defecto
"admin" "password" http.title:"Router"

# Jenkins sin autenticación
http.title:"Dashboard [Jenkins]" -authentication

# GitLab expuesto
http.title:"GitLab" -"Sign in"

# Servidores con DirectoryListing
http.title:"Index of /"

# Memcached expuesto (amplificación DDoS)
port:11211 product:"Memcached"

# SMTP open relay
port:25 "220" "ESMTP"

# Hadoop sin autenticación
port:50070 http.title:"Hadoop"

# Printers expuestas
port:9100 OR port:631 product:printer

# Cobalt Strike C2 (JARM fingerprint)
ssl.jarm:07d14d16d21d21d07c42d41d00041d24a458a375eef0c576d23a7bab9a9fb1
```

### API — endpoints principales

```bash
BASE: https://api.shodan.io

# Información de una IP
GET /shodan/host/{ip}?key=API_KEY

# Búsqueda
GET /shodan/host/search?key=API_KEY&query=QUERY&facets=country

# Contar resultados sin consumir créditos
GET /shodan/host/count?key=API_KEY&query=QUERY

# DNS lookup
GET /dns/resolve?hostnames=google.com&key=API_KEY

# Reverse DNS
GET /dns/reverse?ips=8.8.8.8&key=API_KEY

# Info de tu cuenta
GET /api-info?key=API_KEY
```

### Límites por plan

| Plan               | Búsquedas/mes | Filtros avanzados | `vuln` | Descargas     |
| ------------------ | ------------- | ----------------- | ------ | ------------- |
| Free               | 2 páginas     | ❌                 | ❌      | ❌             |
| Membership (\~$49) | Ilimitadas    | ✅                 | ✅      | 1M resultados |
| Enterprise         | Ilimitadas    | ✅                 | ✅      | Ilimitadas    |

> La mayoría de filtros avanzados requieren al menos cuenta **Membership**.

***

### Referencias

* [Shodan.io](https://www.shodan.io/)
* [Shodan Search Query Fundamentals](https://help.shodan.io/the-basics/search-query-fundamentals)
* [Shodan Filter Reference](https://www.shodan.io/search/filters)
* [Shodan CLI Docs](https://cli.shodan.io/)
* [Shodan API Docs](https://developer.shodan.io/api)
* [FOFA](https://fofa.info/) — alternativa china
* [Censys](https://censys.io/) — alternativa enfocada en certificados
* [ZoomEye](https://www.zoomeye.org/) — otra alternativa
