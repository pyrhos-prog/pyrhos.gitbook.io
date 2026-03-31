# Hydra

Hydra es la herramienta estándar para ataques de fuerza bruta y diccionario contra protocolos de red. Soporta más de 50 servicios y puede paralelizar las peticiones con múltiples threads.

### Sintaxis general

```bash
hydra -l <usuario> -p <password> <host> <servicio>
hydra -L users.txt -P passwords.txt <host> <servicio>
hydra -C credentials.txt <host> <servicio>   # formato user:pass por línea
```

| Flag | Descripción                     |
| ---- | ------------------------------- |
| `-l` | Usuario único                   |
| `-L` | Lista de usuarios               |
| `-p` | Contraseña única                |
| `-P` | Lista de contraseñas            |
| `-C` | Fichero `usuario:contraseña`    |
| `-t` | Threads paralelos (defecto: 16) |
| `-s` | Puerto no estándar              |
| `-f` | Parar al primer éxito           |
| `-v` | Verbose                         |
| `-o` | Guardar resultados en fichero   |

### SSH

```bash
hydra -l root -P rockyou.txt 192.168.1.10 ssh
hydra -L users.txt -P rockyou.txt ssh://192.168.1.10 -t 4
```

> Muchos servidores SSH tienen `MaxAuthTries` bajo (3-6 intentos antes de cortar la conexión). Usar `-t 4` o menos evita que Hydra genere errores de conexión.

### FTP

```bash
hydra -l admin -P rockyou.txt ftp://192.168.1.10
```

### SMB

```bash
hydra -L users.txt -P passwords.txt 192.168.1.10 smb
```

> Para SMB en entornos Windows, es preferible usar `netexec` o `crackmapexec` para evitar lockouts: permiten controlar mejor la tasa de intentos y detectar políticas de bloqueo.

### RDP

```bash
hydra -L users.txt -P passwords.txt rdp://192.168.1.10 -t 4
```

### HTTP/HTTPS — formulario web

Para formularios POST, hay que indicar la URL, los parámetros y la cadena que aparece en caso de fallo:

```bash
hydra -l admin -P rockyou.txt 192.168.1.10 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:Invalid credentials"
```

Para autenticación HTTP Basic:

```bash
hydra -l admin -P rockyou.txt http-get://192.168.1.10/admin
```

### MySQL / MSSQL / PostgreSQL

```bash
hydra -l root -P rockyou.txt 192.168.1.10 mysql
hydra -l sa   -P rockyou.txt 192.168.1.10 mssql
hydra -l postgres -P rockyou.txt 192.168.1.10 postgres
```

### SMTP / IMAP / POP3

```bash
hydra -l user@dominio.com -P rockyou.txt smtp://mail.dominio.com
hydra -l usuario -P rockyou.txt imap://192.168.1.10
```

### Consideraciones operativas

* Empezar con una lista pequeña de contraseñas comunes antes de lanzar rockyou completo.
* En entornos AD con política de lockout, **nunca superar el umbral de intentos fallidos** (típicamente 3-5). Un lockout masivo es detectado inmediatamente y destruye el acceso a cuentas legítimas.
* Añadir delays con `--wait` o `-w` si el servicio implementa rate limiting.
* Hydra no implementa lógica de lockout por sí mismo — es responsabilidad del operador conocer la política antes de atacar.

> En Active Directory, una sola contraseña incorrecta por encima del umbral de lockout bloquea la cuenta. Usar password spraying (ver siguiente página) en lugar de brute-force user-by-user.
