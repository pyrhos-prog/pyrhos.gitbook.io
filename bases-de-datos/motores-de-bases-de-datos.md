---
icon: database
---

# Motores de bases de datos

> Un motor de base de datos (SGBD/DBMS) es el software que implementa el modelo relacional y ejecuta SQL sobre datos reales. Aunque comparten la base de SQL, cada motor tiene particularidades de instalación, sintaxis, tipos de dato y administración que conviene conocer por separado.

### MySQL / MariaDB

El motor open source más extendido, especialmente en aplicaciones web (stack LAMP/LEMP). MariaDB nació como fork de MySQL tras la adquisición de Sun/Oracle, y mantiene compatibilidad casi total.

| Aspecto                   | Detalle                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Puerto por defecto        | 3306                                                                                                           |
| Cliente CLI               | `mysql -u usuario -p`                                                                                          |
| Archivo de configuración  | `/etc/mysql/my.cnf` o `/etc/mysql/mariadb.conf.d/`                                                             |
| Motores de almacenamiento | InnoDB (transaccional, por defecto), MyISAM (sin transacciones, más rápido en lectura)                         |
| Particularidad SQL        | `LIMIT` para paginar, `AUTO_INCREMENT` para claves autoincrementales, backticks (`` ` ``) para identificadores |
| Uso típico en CPTS/HTB    | Muy presente en SQLi de aplicaciones web (WordPress, PHP genérico)                                             |

```sql
SHOW DATABASES;
USE nombre_bd;
SHOW TABLES;
DESCRIBE nombre_tabla;
```

### PostgreSQL

Motor relacional avanzado, más estricto con el estándar SQL y con soporte nativo de tipos de dato complejos (JSON, arrays, tipos geométricos) y extensiones (PostGIS, etc.).

| Aspecto                  | Detalle                                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| Puerto por defecto       | 5432                                                                                                                 |
| Cliente CLI              | `psql -U usuario -d basedatos`                                                                                       |
| Archivo de configuración | `/etc/postgresql/<version>/main/postgresql.conf`                                                                     |
| Autenticación de red     | Controlada aparte en `pg_hba.conf`                                                                                   |
| Particularidad SQL       | `SERIAL`/`IDENTITY` para autoincrementales, `RETURNING` para devolver filas afectadas por `INSERT`/`UPDATE`/`DELETE` |
| Uso típico en CPTS/HTB   | Menos frecuente que MySQL en web, pero habitual en entornos más "serios"/enterprise                                  |

```sql
\l              -- listar bases de datos
\c basedatos    -- conectar a una base de datos
\dt             -- listar tablas
\d tabla        -- describir una tabla
```

### MSSQL (Microsoft SQL Server)

Motor propietario de Microsoft, muy habitual en entornos Windows y Active Directory. Suele encontrarse en infraestructuras corporativas junto a aplicaciones .NET.

| Aspecto                | Detalle                                                                                                                     |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Puerto por defecto     | 1433                                                                                                                        |
| Cliente CLI            | `sqlcmd -S servidor -U usuario -P contraseña`                                                                               |
| Herramienta gráfica    | SQL Server Management Studio (SSMS)                                                                                         |
| Particularidad SQL     | `TOP n` en vez de `LIMIT`, `IDENTITY` para autoincrementales, `GO` como separador de lotes en scripts                       |
| Relevancia ofensiva    | `xp_cmdshell` (si está habilitado) permite ejecución de comandos del sistema desde SQL — vector clásico de post-explotación |
| Uso típico en CPTS/HTB | Frecuente en máquinas orientadas a Active Directory / Windows                                                               |

```sql
SELECT name FROM sys.databases;
USE basedatos;
SELECT * FROM information_schema.tables;
```

### SQLite

Motor embebido, sin proceso de servidor independiente — la base de datos es un único archivo `.db`/`.sqlite`. Habitual en aplicaciones móviles, de escritorio y desarrollo local. No gestiona usuarios/permisos a nivel de motor (el control de acceso depende del sistema de archivos).
