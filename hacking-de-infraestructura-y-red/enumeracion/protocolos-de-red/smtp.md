---
icon: building-magnifying-glass
---

# SMTP

SMTP es un protocolo de red utilizado para **enviar correos electrónicos** en redes IP.\
Opera bajo un modelo **cliente-servidor** y se encarga exclusivamente de la **transmisión y reenvío de emails**, no de su lectura.

#### Puertos Comunes

* **25/tcp** → SMTP estándar (envío entre servidores)
* **587/tcp** → Submission (usuarios autenticados, STARTTLS)
* **465/tcp** → SMTPS (SSL/TLS implícito)

#### Cómo Funciona

1. El cliente (MUA) envía el correo al servidor SMTP.
2. El servidor (MTA) procesa y reenvía el mensaje.
3. El servidor destino lo entrega al buzón mediante un MDA.
4. El usuario lo lee usando **IMAP o POP3**.

Flujo:

Cliente → SMTP → Servidor destino → Buzón → IMAP/POP3

#### Características Técnicas

* Protocolo basado en texto (comandos en claro).
* Puede cifrarse con **TLS (STARTTLS)**.
* Soporta autenticación mediante **SMTP-AUTH**.

#### Riesgos de Seguridad

* Transmisión en texto plano si no se usa TLS.
* Suplantación de identidad (mail spoofing).
* Servidores mal configurados pueden actuar como **Open Relay**.

## Identificación del Servicio

Objetivo:

* Detectar relay abierto
* Enumerar usuarios
* Identificar configuración débil
* Evaluar posibilidad de spoofing

#### Escaneo básico

```bash
nmap -sC -sV -p25 IP
```

#### Enumerar comandos SMTP

```bash
nmap -p25 --script smtp-commands IP
```

#### Detectar Open Relay

```bash
nmap -p25 --script smtp-open-relay -v IP
```

### Enumeración Manual con Telnet

#### Conexión

```bash
telnet IP 25
```

Respuesta esperada: 220 banner

#### Identificación de capacidades

```bash
EHLO test
```

Devuelve:

* PIPELINING
* SIZE
* VRFY
* STARTTLS
* AUTH
* etc.

Si no acepta EHLO:

```bash
HELO test
```

### Enumeración de Usuarios

#### Verificar usuario

```bash
VRFY usuario
```

Posibles respuestas:

* 250 → usuario válido
* 252 → no confirma existencia (común)
* 550 → usuario inexistente

#### Expandir alias

```bash
EXPN lista
```

(No siempre habilitado)

{% hint style="warning" %}
No confiar completamente en VRFY (muchos servidores devuelven 252 siempre).
{% endhint %}

### Prueba de Open Relay Manual

```bash
telnet IP 25
EHLO test
MAIL FROM:<attacker@evil.com>
RCPT TO:<victima@external.com>
```

Si responde: 250 2.1.5 Ok

→ Posible Open Relay.

Si responde: 554 Relay access denied

→ Correctamente configurado.

Configuración vulnerable típica: mynetworks = 0.0.0.0/0

### Envío Manual de Email

```bash
MAIL FROM:<user@domain.com>
RCPT TO:<target@domain.com>
DATA
Subject: test

Mensaje aquí
.
QUIT
```

Finalizar DATA con: .

### Enumeración vía Proxy HTTP

```bash
CONNECT IP:25 HTTP/1.0
```

Útil en entornos restringidos.

### Comandos SMTP

HELO → Inicio sesión&#x20;

EHLO → Inicio extendido (ESMTP)&#x20;

AUTH → Autenticación

MAIL FROM → Remitente&#x20;

RCPT TO → Destinatario&#x20;

DATA → Cuerpo mensaje&#x20;

VRFY → Verifica usuario&#x20;

EXPN → Expande lista&#x20;

RSET → Reset sesión&#x20;

NOOP → Keep-alive&#x20;

QUIT → Cerrar sesión

### Seguridad y Configuración

#### STARTTLS

Activado tras: EHLO STARTTLS

Cifra la conexión.

#### SPF / DKIM / DMARC

Mitigan spoofing.

#### Riesgos comunes

* Open Relay
* VRFY habilitado
* Sin STARTTLS
* AUTH PLAIN sin TLS
* Banner leakage (versión del MTA)
