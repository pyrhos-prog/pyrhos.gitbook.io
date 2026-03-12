---
icon: database
---

# Introducción y conceptos

### ¿Qué es SQL Injection?

SQL Injection (SQLi) es una vulnerabilidad que permite a un atacante interferir con las consultas SQL que una aplicación realiza a su base de datos. Se produce cuando la entrada del usuario se incorpora directamente en una consulta sin la debida sanitización.

```sql
-- Consulta original legítima
SELECT * FROM users WHERE username = 'admin' AND password = '1234';

-- Consulta inyectada
SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything';
```

### ¿Por qué ocurre?

* Concatenación directa de input del usuario en queries SQL
* Falta de prepared statements / consultas parametrizadas
* Validación o sanitización insuficiente del input
* Errores de base de datos expuestos al usuario

### Tipos de SQL Injection

| Tipo                      | Descripción                                 | Respuesta visible |
| ------------------------- | ------------------------------------------- | ----------------- |
| **In-Band > Error-Based** | Extrae datos a través de errores SQL        | Sí                |
| **In-Band > Union-Based** | Extrae datos añadiendo UNION SELECT         | Sí                |
| **Blind > Boolean-Based** | Infiere datos por True/False                | No directamente   |
| **Blind > Time-Based**    | Infiere datos por tiempo de respuesta       | No directamente   |
| **Out-of-Band**           | Exfiltra datos por canal externo (DNS/HTTP) | No directamente   |

### Identificación básica

#### Caracteres de prueba iniciales

```
'
''
`
')
"))
' OR '1'='1
' OR 1=1--
" OR "1"="1
# comentario MySQL
-- comentario SQL
/* comentario */
```

#### Comportamientos que indican SQLi

* Error de base de datos visible (`You have an error in your SQL syntax...`)
* Cambio en el comportamiento de la aplicación (más/menos resultados)
* Respuestas condicionales diferentes (`TRUE` vs `FALSE`)
* Tiempos de respuesta anómalos

#### Prueba rápida

```
Input normal:   https://target.com/item?id=1
Prueba:         https://target.com/item?id=1'
Resultado:      Error SQL → posible SQLi
```

### Contextos de inyección

```sql
-- En cadena de texto (string context)
SELECT * FROM users WHERE name = '[INPUT]'

-- En contexto numérico
SELECT * FROM users WHERE id = [INPUT]

-- En cláusula ORDER BY
SELECT * FROM users ORDER BY [INPUT]

-- En nombre de tabla/columna (no se puede parametrizar)
SELECT [INPUT] FROM users
```

### Bases de datos más comunes y diferencias clave

| Caraterísticas       | MySQL                | PostgreSQL           | MSSQL                | Oracle                      |
| -------------------- | -------------------- | -------------------- | -------------------- | --------------------------- |
| Comentario línea     | `--` / `#`           | `--`                 | `--`                 | `--`                        |
| Comentario bloque    | `/* */`              | `/* */`              | `/* */`              | `/* */`                     |
| Concatenación        | `CONCAT()` / \`      |                      | \`                   | \`                          |
| Versión              | `@@version`          | `version()`          | `@@version`          | `v$version`                 |
| Base de datos actual | `database()`         | `current_database()` | `db_name()`          | `ora_database_name`         |
| Tablas del sistema   | `information_schema` | `information_schema` | `information_schema` | `all_tables`                |
| Sleep                | `SLEEP(n)`           | `pg_sleep(n)`        | `WAITFOR DELAY`      | `dbms_pipe.receive_message` |

### Estructura de information\_schema en MySQL, PostgreSQL y MSSQL

```sql
-- Listar todas las bases de datos
SELECT schema_name FROM information_schema.schemata;

-- Listar tablas de una base de datos
SELECT table_name FROM information_schema.tables WHERE table_schema = 'target_db';

-- Listar columnas de una tabla
SELECT column_name FROM information_schema.columns WHERE table_name = 'users';
```

### Impacto potencial

* **Bypass de autenticación** - acceso como admin sin credenciales
* **Extracción de datos** - credenciales, datos personales, tokens
* **Modificación de datos** - INSERT, UPDATE, DELETE
* **Ejecución de comandos** - si hay xp\_cmdshell, UDF, etc.
* **Lectura y escritura de archivos** - MySQL con FILE privilege

### Claves en SQL injection

> Siempre probar primero con un `'` para confirmar si hay error. Si no hay error visible, pasar a técnicas blind.

> Identificar el gestor de base de datos antes de lanzar payloads específicos. Un payload MySQL no funcionará en Oracle.

> Revisar siempre los parámetros GET, POST, cookies, headers (User-Agent, Referer, X-Forwarded-For).
