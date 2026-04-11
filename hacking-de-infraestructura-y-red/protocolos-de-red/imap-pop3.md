# IMAP / POP3

***

IMAP y POP3 son los protocolos que los clientes de correo usan para **recibir** y leer emails desde un servidor. A diferencia de SMTP (que envía), estos protocolos permiten acceder al buzón. Son especialmente relevantes en pentests porque a veces contienen información sensible y credenciales válidas.

### Puertos

| Protocolo | Puerto | Descripción        |
| --------- | ------ | ------------------ |
| **IMAP**  | 143    | IMAP sin cifrar    |
| **IMAPS** | 993    | IMAP sobre SSL/TLS |
| **POP3**  | 110    | POP3 sin cifrar    |
| **POP3S** | 995    | POP3 sobre SSL/TLS |

#### IMAP vs POP3

|                            | IMAP                                 | POP3                                           |
| -------------------------- | ------------------------------------ | ---------------------------------------------- |
| **Sincronización**         | Los emails permanecen en el servidor | Los emails se descargan y borran del servidor  |
| **Múltiples dispositivos** | Sí, todos ven los mismos emails      | No, cada dispositivo descarga emails distintos |
| **Carpetas**               | Soporte completo de carpetas         | Sin carpetas                                   |
| **Uso actual**             | El estándar moderno                  | Legacy, cada vez menos usado                   |

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sV -p 110,143,993,995 target

# Scripts específicos
nmap -sC -sV -p 110,143 target
nmap --script imap-capabilities,imap-brute -p 143 target
nmap --script pop3-capabilities,pop3-brute -p 110 target
```

#### Conexión manual con netcat

Conectar directamente a IMAP o POP3 permite ver las capacidades del servidor y probar autenticación. El banner revela el software del servidor.

**IMAP manual:**

```bash
nc -nv target 143
# Banner: * OK Dovecot ready.

# Listar capacidades
A1 CAPABILITY

# Login
A2 LOGIN usuario contraseña

# Listar carpetas/buzones
A3 LIST "" "*"

# Seleccionar la bandeja de entrada
A4 SELECT INBOX

# Ver los primeros correos
A5 FETCH 1 (FLAGS BODY[HEADER])

# Leer un email completo
A6 FETCH 1 BODY[]

# Buscar emails con una palabra clave
A7 SEARCH TEXT "contraseña"

# Salir
A8 LOGOUT
```

**POP3 manual:**

```bash
nc -nv target 110
# Banner: +OK Dovecot ready.

# Ver capacidades
CAPA

# Login
USER usuario
PASS contraseña

# Listar emails
LIST

# Ver un email
RETR 1

# Borrar un email
DELE 1

# Salir
QUIT
```

#### Conexión con SSL

```bash
# IMAPS (puerto 993)
openssl s_client -connect target:993

# POP3S (puerto 995)
openssl s_client -connect target:995

# Una vez conectado, los comandos son los mismos que en la versión sin cifrar
```

#### Brute force de credenciales

```bash
# Hydra para IMAP
hydra -l usuario -P /usr/share/wordlists/rockyou.txt imap://target
hydra -L users.txt -P passwords.txt imap://target

# Hydra para POP3
hydra -l usuario -P /usr/share/wordlists/rockyou.txt pop3://target

# Con SSL
hydra -l usuario -P /usr/share/wordlists/rockyou.txt -S imaps://target
hydra -l usuario -P /usr/share/wordlists/rockyou.txt -S pop3s://target
```

### Qué buscar una vez autenticado

Si se obtienen credenciales válidas (por brute force, credential stuffing o por haberlas encontrado en otro servicio), el contenido de los buzones puede ser muy valioso:

* Credenciales de otros servicios enviadas por email (bienvenida, reset de contraseña)
* Documentos adjuntos con información sensible
* Comunicaciones internas con datos de la infraestructura
* Tokens de API o claves hardcodeadas en emails de notificación

```bash
# Con IMAP: buscar emails con palabras clave sensibles
A7 SEARCH TEXT "password"
A8 SEARCH TEXT "contraseña"
A9 SEARCH TEXT "api_key"
A10 SEARCH TEXT "token"
A11 SEARCH FROM "no-reply@"    # emails automáticos de sistemas
```

### Riesgos y misconfiguraciones

| Riesgo                              | Descripción                                                                                       |
| ----------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Credenciales en claro**           | POP3/IMAP en puertos 110/143 transmiten usuario y contraseña sin cifrar. Capturables con MITM.    |
| **Brute force sin rate limiting**   | Sin protección contra intentos repetidos, es posible bruteforcear credenciales.                   |
| **Reutilización de contraseñas**    | Las credenciales de email suelen reutilizarse en otros servicios. Probar en VPN, OWA, etc.        |
| **Información sensible en buzones** | Emails de bienvenida con contraseñas temporales, notificaciones con tokens, documentos adjuntos.  |
| **Enumeración de usuarios**         | Algunos servidores diferencian en la respuesta entre usuario inexistente y contraseña incorrecta. |
| **Acceso a OWA/Webmail**            | Si hay Outlook Web Access o Roundcube expuestos, las mismas credenciales permiten acceso web.     |

> Si se consiguen credenciales de email válidas en un entorno corporativo con **Microsoft Exchange**, el puerto 443 probablemente tenga **OWA (Outlook Web Access)** — las mismas credenciales dan acceso al correo web y potencialmente al entorno de Microsoft 365 completo.

> La **reutilización de contraseñas** hace que unas credenciales de IMAP/POP3 sean mucho más valiosas de lo que parecen. Probar siempre las credenciales encontradas en SMB, SSH, VPN, paneles web y cualquier otro servicio identificado en el target.
