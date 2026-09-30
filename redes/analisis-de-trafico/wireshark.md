---
icon: ethernet
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Wireshark

## Wireshark

### ¿Que es Wireshark?

Wireshark es un programa que sirve para analizar protocolos de red que permite capturar y analizar el tráfico de una red.

<div data-with-frame="true"><figure><img src="https://1680330859-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fz0iyBve0fcLSJpEvb9YL%2Fuploads%2Fa5IsUdwLUjKj52KIa0vI%2Fimage.png?alt=media&#x26;token=618c82b0-c299-4292-83ce-0ee18fd13fde" alt=""><figcaption></figcaption></figure></div>

**Wireshark es una herramienta muy útil para:**

* Diagnosticar problemas de red, como lentitud o pérdidas de paquetes.
* Aprender cómo funcionan los diferentes protocolos.
* Detectar tráfico sospechoso.
* Comprobar el funcionamiento de configuraciones de red o nuevos servicios.<br>

### Operadores Lógicos de Referencia

| Operador   | Alternativa | Descripción                                |
| ---------- | ----------- | ------------------------------------------ |
| `==`       | `eq`        | Igual                                      |
| `!=`       | `ne`        | Diferente                                  |
| `>` / `<`  | `gt` / `lt` | Mayor que / Menor que                      |
| `&&`       | `and`       | Y lógico                                   |
| `\|\|`     | `or`        | O lógico                                   |
| `!`        | `not`       | Negación                                   |
| `contains` | -           | Coincidencia de subcadena (case-sensitive) |
| `matches`  | `~`         | Expresión regular (Perl regex compatible)  |

## Wireshark para ciberseguridad

### Filtros de Captura

```
# Tráfico de un host o subred específica
host 10.10.10.5
net 192.168.1.0/24

# Excluir tráfico propio para no saturar (ej. sesión SSH del auditor)
not host 192.168.1.150 and not port 22

# Capturar solo puertos específicos
port 80 or port 443 or port 445 or port 88

# Capturar solo paquetes SYN (escaneo / nuevos handshakes)
tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0

# Broadcasts y multicasts locales (LLMNR, mDNS, ARP)
ether multicast or ether broadcast
```

### Reconocimiento y Descubrimiento de Red

#### Escaneos de Puertos&#x20;

```
# Detectar escaneos SYN (SYN enviado sin ACK posterior)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Escaneos TCP Connect completados rápidamente
tcp.flags.syn == 1 and tcp.flags.ack == 1

# Escaneos Null, Xmas o FIN
tcp.flags == 0x000                                 # NULL Scan
tcp.flags.fin == 1 and tcp.flags.urg == 1 and tcp.flags.push == 1  # Xmas Scan
tcp.flags.fin == 1 and tcp.flags.ack == 0          # FIN Scan

# Detección de barridos de puertos por host origen
ip.src == <IP_ATACANTE> && tcp
```

#### Tráfico de Broadcast  o envenenamiento de Capa 2 (LLMNR / NBT-NS / mDNS / ARP)

Filtros esenciales para evaluar viabilidad de ataques tipo Responder y ARP Spoofing:

```
# Envenenamiento de resolución de nombres (Responder targets)
udp.port == 5355                                   # LLMNR
udp.port == 137                                    # NetBIOS Name Service (NBT-NS)
udp.port == 5353                                   # mDNS
llmnr || nbns || mdns

# Peticiones y respuestas ARP (Detección de ARP Spoofing / Gratuitous ARP)
arp
arp.duplicate-address-frame                        # Alertas de suplantación/conflicto IP
arp.opcode == 2                                    # Respuestas ARP
```

### Credenciales y Protocolos en Texto Claro

#### HTTP & Formularios Web

```
# Métodos POST (frecuente envío de contraseñas/tokens)
http.request.method == "POST"

# Búsqueda de parámetros clave en URI o payload
http.request.uri contains "login" || http.request.uri contains "admin"
http contains "password" || http contains "user" || http contains "passwd"

# Cookies y cabeceras de autorización
http.cookie || http.authorization
```

#### Protocolos Heredados&#x20;

```
# FTP (User y Pass)
ftp.request.command == "USER" || ftp.request.command == "PASS"

# Telnet (sesiones interactivas sin cifrado)
telnet

# SMTP / IMAP / POP3 (Autenticación de correo)
smtp.req.command == "AUTH" || pop.request.parameter contains "PASS"
imap.request contains "LOGIN"

# SNMP (Comunidades públicas/por defecto)
snmp.version == 0 || snmp.version == 1             # SNMPv1 / SNMPv2c
snmp.community contains "public" || snmp.community contains "private"
```

### Active Directory y Pivoting Interno

#### Kerberoasting, AS-REP Roasting, Pass-the-Ticket

```
# Tráfico general Kerberos
kerberos

# Peticiones AS-REQ (Útil para detectar AS-REP Roasting / Cuentas sin pre-auth)
kerberos.msg_type == 10

# Peticiones TGS-REQ (Kerberoasting - Solicitud de SPNs)
kerberos.msg_type == 12

# Identificar fallos de autenticación (fuerza bruta / user enumeration)
kerberos.error_code == 6                           # KDC_ERR_C_PRINCIPAL_UNKNOWN
kerberos.error_code == 18                          # KDC_ERR_PREAUTH_FAILED
kerberos.error_code == 24                          # KDC_ERR_PREAUTH_EXPIRED
```

#### SMB / RPC / NTLM (Pass-the-Hash, Relay attacks)

```
# Tráfico SMB general
smb || smb2

# Autenticación NTLMSSP (extracción de NTLMv2 hashes de la captura)
ntlmssp
ntlmssp.messagetype == 1                           # NTLMSSP_NEGOTIATE
ntlmssp.messagetype == 2                           # NTLMSSP_CHALLENGE
ntlmssp.messagetype == 3                           # NTLMSSP_AUTH

# Intentos de acceso a recursos compartidos (IPC$, C$, ADMIN$)
smb2.tree contains "IPC$" || smb2.tree contains "ADMIN$" || smb2.tree contains "C$"

# Tráfico DCE/RPC (MS-RPC, PsExec, Samr, Lsar)
dcerpc
```

### Canales Encubiertos y C2

#### DNS Tunneling & Exfiltración

```
# Búsqueda de consultas con payloads largos (posible base64 / hex en subdominio)
dns.flags.response == 0 and dns.qry.name.len > 40

# Tipos de registros sospechosos para C2
dns.qry.type == 16                                 # Consultas TXT (C2 payload staging)
dns.qry.type == 1                                  # Consultas A

# Respuestas NXDOMAIN masivas (DGA - Domain Generation Algorithms)
dns.flags.rcode == 3
```

#### ICMP Tunneling

```
# Paquetes ICMP Echo con payloads de datos anormales (mayor al ping estándar de 32/64 bytes)
icmp.type == 8 and data.len > 64
```

#### TLS / HTTPS (C2 Hunting & Beaconing)

```
# Handshakes TLS (extracción de Server Name Indication)
tls.handshake.extension.type == 0                  # SNI extension
tls.handshake.extensions_server_name contains "dominio-c2.com"

# Tráfico TLS a puertos no estándar
tls && !(tcp.port == 443 || tcp.port == 8443)
```

### Extracción de Archivos y Artefactos

1. **Objetos HTTP / SMB / IMF:**
   * Menú superior: `File` $\rightarrow$ `Export Objects` $\rightarrow$ Seleccionar `HTTP`, `SMB`, o `IMF`.
   * Permite guardar directamente payloads transferidos, binarios `.exe`, documentos `.docx`, scripts `.ps1`, etc.
2. **Reconstrucción de streams completos:**
   * Click derecho sobre cualquier paquete $\rightarrow$ `Follow` $\rightarrow$ `TCP Stream` (o `HTTP / TLS Stream`).
   * Cambiar visualización a **ASCII**, **Hexdump** o **Raw** para exportar binarios o shellcodes.

### TShark

Cuando no tienes entorno gráfico en la máquina comprometida o servidor de salto:

```bash
# 1. Extraer credenciales HTTP POST en vivo
tshark -i eth0 -Y 'http.request.method == "POST"' -T fields -e ip.src -e http.host -e http.file_data

# 2. Extraer dominios consultados por DNS en tiempo real
tshark -i eth0 -f "udp port 53" -T fields -e ip.src -e dns.qry.name

# 3. Extraer nombres de usuario NTLMv2 autenticándose
tshark -r capture.pcap -Y 'ntlmssp.messagetype == 3' -T fields -e ip.src -e ntlmssp.auth.username -e ntlmssp.auth.domain

# 4. Exportar objetos HTTP de una captura PCAP
tshark -r capture.pcap --export-objects "http,./extracted_files/"

# 5. Listar IPs con más conexiones (Top Talkers)
tshark -r capture.pcap -q -z conv,ip
```

{% file src="../../.gitbook/assets/CheatSheet_Wireshark.pdf" %}
