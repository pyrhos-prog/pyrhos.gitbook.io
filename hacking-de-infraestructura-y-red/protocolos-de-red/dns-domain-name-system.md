# DNS — Domain Name System

DNS (Domain Name System) es el protocolo que traduce nombres de dominio a direcciones IP. Opera en el puerto **53** (UDP para consultas normales, TCP para transferencias de zona y respuestas grandes). Aunque su función principal es la resolución de nombres, DNS es una fuente de información enorme para el reconocimiento pasivo y activo.

### Tipos de registros DNS

| Registro  | Descripción                                                    |
| --------- | -------------------------------------------------------------- |
| **A**     | IPv4 → dirección IP de un hostname                             |
| **AAAA**  | IPv6 → dirección IPv6 de un hostname                           |
| **CNAME** | Canonical Name → alias que apunta a otro nombre                |
| **MX**    | Mail Exchange → servidores de correo del dominio               |
| **NS**    | Name Server → servidores DNS autoritativos del dominio         |
| **TXT**   | Texto → verificaciones (SPF, DKIM, DMARC), información variada |
| **PTR**   | Pointer → resolución inversa (IP → nombre)                     |
| **SOA**   | Start of Authority → información del servidor DNS principal    |
| **SRV**   | Service → ubicación de servicios específicos (SIP, LDAP...)    |
| **AXFR**  | Zone Transfer → copia completa de la zona DNS                  |

### Enumeración

#### Consultas DNS básicas con dig

`dig` es la herramienta principal para consultas DNS. Es más detallada y flexible que `nslookup`.

```bash
# Consulta A (IP del dominio)
dig target.com

# Consulta tipo específico
dig target.com MX
dig target.com NS
dig target.com TXT
dig target.com SOA
dig target.com AAAA

# Consulta a un servidor DNS específico
dig @8.8.8.8 target.com
dig @ns1.target.com target.com

# Resolución inversa (IP → nombre)
dig -x 1.2.3.4

# Respuesta corta (solo el resultado)
dig target.com +short

# Todos los registros disponibles
dig target.com ANY
```

#### Transferencia de zona (AXFR)

Una transferencia de zona es una copia completa de todos los registros DNS de un dominio. Debería estar restringida solo a los servidores DNS secundarios, pero a veces está mal configurada y permite a cualquiera obtener el mapa completo de la infraestructura.

```bash
# Intentar transferencia de zona
dig axfr target.com @ns1.target.com
dig axfr target.com @ns2.target.com

# Con host
host -l target.com ns1.target.com

# Con nslookup
nslookup
> server ns1.target.com
> set type=any
> ls -d target.com
```

Si tiene éxito, la transferencia de zona revela **todos los subdominios, IPs internas, nombres de hosts y servicios** del dominio de una sola vez.

#### Enumeración de subdominios

```bash
# dnsenum — enumeración completa + transferencia de zona + brute force
dnsenum target.com
dnsenum --dnsserver ns1.target.com target.com

# dnsrecon — muy completo
dnsrecon -d target.com                          # enumeración estándar
dnsrecon -d target.com -t axfr                  # solo transferencia de zona
dnsrecon -d target.com -t brt -D subdomains.txt # brute force de subdominios

# fierce — descubrimiento de subdominios y IPs
fierce --domain target.com
fierce --domain target.com --wordlist subdomains.txt

# subfinder (pasivo, sin tocar el target)
subfinder -d target.com
subfinder -d target.com -o subdominios.txt

# amass (el más completo)
amass enum -d target.com
amass enum -passive -d target.com     # solo fuentes pasivas
amass enum -active -d target.com      # incluyendo brute force
```

#### Brute force de subdominios

```bash
# gobuster en modo DNS
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# ffuf
ffuf -u http://FUZZ.target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Con resolución de IPs
gobuster dns -d target.com -w wordlist.txt -i    # -i muestra las IPs
```

#### Nmap para DNS

```bash
# Detección del servidor DNS
nmap -sV -p 53 target

# Scripts de nmap para DNS
nmap --script dns-brute target
nmap --script dns-zone-transfer --script-args dns-zone-transfer.domain=target.com -p 53 target
nmap --script dns-recursion target     # ver si el servidor permite recursión abierta
```

### Fuentes de información pasiva

Antes de lanzar cualquier herramienta activa, estas fuentes revelan subdominios sin tocar el objetivo:

* **Certificate Transparency Logs** — `crt.sh`: `https://crt.sh/?q=%.target.com`
* **Wayback Machine / GAU** — URLs históricas con subdominios: `echo "target.com" | gau`
* **Google Dorks** — `site:target.com -www` para descubrir subdominios indexados
* **Shodan** — buscar `hostname:target.com` revela IPs y puertos de sus hosts
* **VirusTotal** — `https://virustotal.com/gui/domain/target.com/relations` muestra subdominios conocidos

### Riesgos y misconfiguraciones

| Riesgo                                     | Descripción                                                                                                                         |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Transferencia de zona abierta**          | Permite obtener todos los registros DNS del dominio de una vez. Mapa completo de la infraestructura.                                |
| **DNS Recursion abierta**                  | El servidor responde a consultas de cualquier IP, puede usarse para ataques DDoS de amplificación.                                  |
| **Subdomain Takeover**                     | Un subdominio apunta (CNAME) a un servicio externo que ya no existe o cuya cuenta ha sido abandonada. El atacante puede reclamarlo. |
| **Registros TXT con información sensible** | SPF, DKIM, DMARC mal configurados revelan proveedores de email e infraestructura.                                                   |
| **Wildcard DNS mal configurado**           | Resuelve cualquier subdominio a la misma IP, dificultando la detección pero potencialmente exponiendo servicios.                    |
| **Zona de transferencia interna expuesta** | En redes corporativas, el servidor DNS interno puede revelar IPs privadas, nombres de servidores internos y arquitectura de red.    |

> Una **transferencia de zona exitosa** es uno de los mejores hallazgos de reconocimiento posibles: en un solo comando se obtiene el mapa completo de todos los hosts, IPs y servicios del dominio.

> Los registros **TXT** son muy informativos. Un registro SPF como `v=spf1 include:sendgrid.net include:mailchimp.com ~all` revela exactamente qué proveedores de email usa la empresa, útil para ataques de phishing dirigido.
