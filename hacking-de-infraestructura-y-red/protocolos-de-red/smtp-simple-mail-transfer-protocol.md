# SMTP — Simple Mail Transfer Protocol

SMTP es el protocolo estándar para el envío de correo electrónico entre servidores. Opera en los puertos **25** (comunicación entre servidores), **587** (submission con autenticación, clientes de correo) y **465** (SMTPS, SMTP sobre SSL). Puerto 25 es el más relevante en pentests de infraestructura.

### Puertos y usos

| Puerto  | Uso                                | Notas                                                                                        |
| ------- | ---------------------------------- | -------------------------------------------------------------------------------------------- |
| **25**  | SMTP entre servidores (MTA)        | Suele estar abierto en servidores de correo. Rara vez acepta conexiones de clientes finales. |
| **587** | Submission (clientes con STARTTLS) | Para clientes de correo con autenticación.                                                   |
| **465** | SMTPS (SSL/TLS implícito)          | Cifrado desde el inicio de la conexión.                                                      |

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sV -p 25,587,465 target

# Scripts específicos para SMTP
nmap -sC -sV -p 25 target
nmap --script smtp-commands,smtp-enum-users,smtp-open-relay -p 25 target
nmap --script smtp-vuln* -p 25 target
```

#### Conexión manual con netcat / telnet

Conectar directamente al servidor SMTP permite ver el banner (que revela el software y versión) y probar comandos manualmente:

```bash
# Conexión al puerto 25
nc -nv target 25
telnet target 25
```

Una vez conectado, la comunicación SMTP sigue este flujo:

```
220 mail.target.com ESMTP Postfix (Ubuntu)    ← Banner del servidor

EHLO attacker.com                              ← Saludar al servidor (Extended HELO)
250-mail.target.com                            ← Respuesta con capacidades
250-SIZE 10240000
250-VRFY                                       ← VRFY habilitado → enumeración de usuarios
250-EXPN                                       ← EXPN habilitado → expandir listas
250 HELP

VRFY root                                      ← Verificar si existe el usuario root
252 2.0.0 root                                 ← Usuario existe

VRFY noexiste                                  ← Usuario que no existe
550 5.1.1 <noexiste>: Recipient address rejected

QUIT
```

#### Enumeración de usuarios — VRFY, EXPN, RCPT TO

SMTP tiene tres comandos que pueden revelar si un usuario existe en el servidor:

**VRFY** — Verifica si una dirección existe. Si está habilitado, confirma usuarios directamente.

**EXPN** — Expande una lista de distribución, revelando todos sus miembros.

**RCPT TO** — Intento de envío. La diferencia en la respuesta entre un usuario que existe y uno que no revela información de enumeración.

```bash
# Manual con netcat
nc target 25
EHLO test
VRFY admin
VRFY root
VRFY info

# Automatizado con smtp-user-enum
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t target
smtp-user-enum -M EXPN -U users.txt -t target
smtp-user-enum -M RCPT -U users.txt -D target.com -t target

# Opciones:
# -M VRFY/EXPN/RCPT TO → método de enumeración
# -U → wordlist de usuarios
# -D → dominio para construir direcciones email
# -t → target IP
```

#### Open Relay

Un **open relay** es un servidor SMTP que acepta y retransmite correo de cualquier origen a cualquier destino sin autenticación. Era común en servidores mal configurados y permite enviar spam o emails de phishing usando el servidor de la víctima como relay.

```bash
# Probar si el servidor es open relay
nmap --script smtp-open-relay -p 25 target

# Manual — intentar enviar correo de terceros a terceros
nc target 25
EHLO test.com
MAIL FROM: <externo@gmail.com>
RCPT TO: <victima@empresa.com>
DATA
Subject: Test relay
Este es un test de open relay.
.
QUIT

# Si responde 250 OK → open relay confirmado
```

#### SMTP con STARTTLS

```bash
# Conectar con openssl para ver el certificado y comunicarse cifrado
openssl s_client -starttls smtp -connect target:25

# Directo a SMTPS (puerto 465)
openssl s_client -connect target:465
```

### Riesgos y misconfiguraciones

| Riesgo                                   | Descripción                                                                                               |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **VRFY habilitado**                      | Permite enumerar usuarios válidos del sistema. Brute force con wordlist revela cuentas existentes.        |
| **EXPN habilitado**                      | Revela los miembros de listas de distribución.                                                            |
| **Open Relay**                           | El servidor retransmite correo sin autenticación → spam, phishing usando la reputación del servidor.      |
| **Banner informativo**                   | El banner SMTP revela software, versión y a veces el hostname interno. Información útil para buscar CVEs. |
| **Credenciales débiles en port 587**     | Brute force en el puerto de submission autenticado.                                                       |
| **Sin cifrado (puerto 25 sin STARTTLS)** | Credenciales y contenido del correo viajan en claro.                                                      |

> La enumeración de usuarios via SMTP es especialmente útil antes de un ataque de **password spraying**: si se sabe exactamente qué cuentas existen, se puede hacer spraying dirigido evitando cuentas que no existen y reduciendo el ruido.

> El banner del servidor SMTP revela el MTA (Mail Transfer Agent) y su versión. Postfix, Exim, Sendmail y Microsoft Exchange tienen CVEs documentados en versiones específicas. Siempre buscar la versión en Exploit-DB o `searchsploit` tras identificarla.
