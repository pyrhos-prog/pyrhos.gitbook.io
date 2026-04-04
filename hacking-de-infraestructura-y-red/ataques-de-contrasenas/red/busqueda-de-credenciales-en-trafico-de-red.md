# Búsqueda de Credenciales en Tráfico de Red

La mayoría de aplicaciones modernas usan TLS para cifrar el tráfico, pero los entornos reales siempre tienen excepciones: sistemas legacy, servicios mal configurados o aplicaciones internas sin HTTPS. Estos gaps permiten capturar credenciales en texto claro directamente del tráfico de red.

### Protocolos sin cifrar vs. sus alternativas seguras

| Protocolo inseguro | Alternativa cifrada  | Uso                            |
| ------------------ | -------------------- | ------------------------------ |
| HTTP               | HTTPS                | Transferencia web              |
| FTP                | FTPS / SFTP          | Transferencia de archivos      |
| SNMP v1/v2c        | SNMPv3 (con cifrado) | Gestión de dispositivos de red |
| POP3               | POP3S                | Recuperación de email          |
| IMAP               | IMAPS                | Acceso a email en servidor     |
| SMTP               | SMTPS                | Envío de email                 |
| LDAP               | LDAPS                | Consultas de directorio (AD)   |
| RDP                | RDP con TLS          | Escritorio remoto Windows      |
| SMB v1/v2          | SMB 3.0 sobre TLS    | Compartición de archivos       |
| VNC                | VNC con TLS/SSL      | Control remoto gráfico         |
| DNS                | DNS over HTTPS (DoH) | Resolución de nombres          |

> En redes internas corporativas es frecuente encontrar SNMP v1/v2c, FTP y HTTP activos en dispositivos de red, impresoras y sistemas legacy que nunca fueron migrados a sus alternativas cifradas.

### Wireshark

Wireshark es el analizador de paquetes estándar, incluido por defecto en la mayoría de distribuciones de pentesting. Permite analizar capturas `.pcap`/`.pcapng` o tráfico en vivo.

#### Filtros de display útiles

| Filtro                                           | Descripción                                       |
| ------------------------------------------------ | ------------------------------------------------- |
| `http`                                           | Todo el tráfico HTTP                              |
| `http.request.method == "POST"`                  | Peticiones POST — formularios de login sin cifrar |
| `http contains "passw"`                          | Paquetes HTTP con la cadena "passw"               |
| `ftp`                                            | Tráfico FTP (credenciales en claro)               |
| `snmp`                                           | Tráfico SNMP — community strings visibles         |
| `dns`                                            | Resoluciones DNS                                  |
| `tcp.port == 21`                                 | FTP control channel                               |
| `tcp.port == 23`                                 | Telnet                                            |
| `ip.addr == 192.168.1.10`                        | Tráfico de un host específico                     |
| `ip.src == 192.168.1.5 && ip.dst == 192.168.1.1` | Tráfico entre dos hosts                           |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0`       | Paquetes SYN — detección de escaneos              |
| `tcp.stream eq 53`                               | Seguir una conversación TCP específica            |
| `eth.addr == 00:11:22:33:44:55`                  | Tráfico de/hacia una MAC concreta                 |

#### Buscar credenciales en HTTP

Para localizar paquetes con contenido sensible en HTTP:

* Filtro directo: `http contains "passw"` o `http contains "password"`
* Búsqueda manual: `Edit → Find Packet` → String → `passw`
* Seguir el stream de un POST sospechoso: click derecho → `Follow → TCP Stream`

Los formularios de login enviados por HTTP sin cifrar exponen usuario y contraseña en el campo `application/x-www-form-urlencoded` del cuerpo del paquete POST.

#### Buscar community strings SNMP

```
snmp
```

En el panel de detalles del paquete, el campo `community` contiene la community string en texto claro. Valores como `public`, `private` o strings personalizadas son visibles directamente.

### Pcredz

Pcredz automatiza la extracción de credenciales de capturas o tráfico en vivo sin necesidad de filtrar manualmente en Wireshark.

#### Qué extrae

* Community strings SNMP (v1/v2c)
* Credenciales FTP, POP3, SMTP, IMAP en texto claro
* Credenciales de formularios HTTP (Basic auth + POST forms)
* Hashes NTLMv1/v2 de tráfico SMB, LDAP, MSSQL, DCE-RPC, HTTP
* Hashes Kerberos AS-REQ (etype 23) — crackeables con Hashcat `-m 18200`
* Números de tarjeta de crédito

#### Instalación

```bash
# Clonar repositorio e instalar dependencias
git clone https://github.com/lgandx/PCredz
cd PCredz
pip install -r requirements.txt

# Alternativa: Docker (ver README del repositorio)
```

#### Uso contra un archivo de captura

```bash
./Pcredz -f captura.pcapng -t -v
```

Salida de ejemplo:

```
[*] Found SNMPv2 Community string: s3cr3tStr1ng

[*] FTP User: ltnbob
[*] FTP Pass: qwerty123

[*] NTLMv2 hash: Administrator::WORKGROUP:...
```

#### Uso contra interfaz en vivo (requiere root)

```bash
sudo ./Pcredz -i eth0 -t -v
```

| Flag | Descripción                              |
| ---- | ---------------------------------------- |
| `-f` | Archivo de captura `.pcap`/`.pcapng`     |
| `-i` | Interfaz de red para captura en vivo     |
| `-t` | Activar escaneo de números de tarjeta    |
| `-v` | Verbose — muestra detalles del procesado |

> La captura de tráfico en vivo en redes conmutadas solo recibe el tráfico destinado al propio host o broadcast. Para capturar tráfico de otros hosts se necesita ARP spoofing, port mirroring (SPAN) o acceso físico a un punto de agregación de red.

### Flujo típico en un pentest interno

Con acceso a un segmento de red interno, el proceso habitual es:

1. Identificar protocolos sin cifrar con un escaneo Nmap (`-sV` sobre puertos 21, 23, 25, 80, 110, 143, 161, 389\`)
2. Capturar tráfico durante un tiempo razonable con `tcpdump` o Wireshark
3. Analizar la captura con Pcredz para extracción automática
4. Revisar manualmente en Wireshark los streams de protocolos de interés

```bash
# Captura rápida con tcpdump
sudo tcpdump -i eth0 -w captura.pcapng

# Procesar con Pcredz
./Pcredz -f captura.pcapng -v
```
