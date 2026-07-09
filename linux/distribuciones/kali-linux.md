---
icon: debian
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
---

# Kali Linux

### ¿Que es Kali Linux?

Kali linux es una distribución basada en [Debian](https://pyrhos.gitbook.io/pyrhos-wiki/linux/distribuciones/debian), mantenida por **Offensive Security,** diseñada para **seguridad informática**, **auditorías**, **análisis forense digital** y **pruebas de penetración (pentesting).**

<div data-with-frame="true"><figure><img src="https://36118879-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F5YP0HIeDL3z0KLMojf1y%2Fuploads%2FqCalMwPeT29DmiR9yc7h%2Fimage.png?alt=media&#x26;token=c0a71394-7ff0-4969-b890-7e7d60ac9291" alt=""><figcaption></figcaption></figure></div>

### Datos clave

| Campo                      | Valor                       |
| -------------------------- | --------------------------- |
| **Base**                   | Debian testing              |
| **Init system**            | systemd                     |
| **Escritorio por defecto** | XFCE                        |
| **Ciclo**                  | Rolling release             |
| **Empresa**                | Offensive Security (OffSec) |
| **Usuario por defecto**    | `kali`                      |

> Kali está diseñada para un propósito específico. No es una distro de uso general: viene con servicios de red desactivados por defecto, muchas herramientas requieren privilegios elevados, y no tiene las mismas consideraciones de estabilidad que Debian stable.&#x20;



### Formatos de Kali disponibles

| Formato                  | Uso                                             |
| ------------------------ | ----------------------------------------------- |
| **ISO / OVA para VM**    | Uso más habitual — VirtualBox / VMware          |
| **WSL2**                 | Kali en Windows Subsystem for Linux             |
| **USB booteable**        | Auditorías en el cliente                        |
| **USB con persistencia** | USB booteable que guarda cambios entre sesiones |
| **NetHunter**            | Kali en Android (dispositivos rooteados)        |
| **Raspberry Pi**         | Imagen específica disponible                    |

### Diferencias con Debian

Lo que comparten: APT, dpkg, `.deb`, systemd, estructura de directorios.

Lo que diferencia a Kali:

* Más de 600 herramientas de seguridad preinstaladas
* Metapaquetes por categoría de pentest
* Kernel con algunos patches y drivers Wi-Fi adicionales para pentest
* Servicios de red desactivados por defecto
* Repositorio `kali-rolling` en lugar de `bookworm`

### Gestor de paquetes — APT

Kali hereda APT de Debian. El uso es idéntico, con la diferencia de que los repositorios apuntan a los servidores de Kali.

```bash
# Actualizar el sistema (hacer SIEMPRE antes de usar herramientas)
sudo apt update && sudo apt full-upgrade -y

# Instalar / eliminar
sudo apt install nmap
sudo apt purge herramienta && sudo apt autoremove

# Buscar
apt search nmap
apt show metasploit-framework
```

> Actualizar Kali con frecuencia es crítico, las herramientas de seguridad reciben actualizaciones constantes con nuevas técnicas y exploits.

#### Repositorios de Kali

El repositorio principal es `kali-rolling`. El `sources.list` por defecto contiene:

```
deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware
```

Kali también ofrece `kali-bleeding-edge` para las versiones más experimentales:

```bash
echo "deb http://http.kali.org/kali kali-bleeding-edge main" | \
    sudo tee /etc/apt/sources.list.d/kali-bleeding-edge.list
sudo apt update
```

### Gestión de servicios

Kali no inicia servicios de red automáticamente por defecto. Hay que activarlos manualmente cuando se necesiten:

```bash
# SSH
sudo systemctl start ssh
sudo systemctl enable ssh       # iniciar automáticamente en el arranque

# PostgreSQL + Metasploit
sudo systemctl start postgresql
sudo msfdb init && msfconsole

# Apache (para servir payloads o archivos)
sudo systemctl start apache2
# Archivos en /var/www/html/
```

> Cambiar siempre la contraseña por defecto antes de activar SSH. Las credenciales `kali:kali` son públicamente conocidas. Ejecutar `passwd` para cambiarla.

### Metapaquetes — instalar colecciones de herramientas

Kali organiza sus herramientas en **metapaquetes** por categoría, permitiendo instalar solo lo que se necesita sin descargar los \~20GB del conjunto completo.

| Metapaquete                        | Contenido                           |
| ---------------------------------- | ----------------------------------- |
| `kali-tools-top10`                 | Las 10 herramientas más usadas      |
| `kali-linux-default`               | Instalación estándar de Kali        |
| `kali-linux-large`                 | Instalación amplia                  |
| `kali-linux-everything`            | Todas las herramientas (\~20GB)     |
| `kali-tools-web`                   | Burp Suite, sqlmap, nikto, ffuf...  |
| `kali-tools-wireless`              | Aircrack-ng, reaver, hostapd-wpe... |
| `kali-tools-exploitation`          | Metasploit, exploits...             |
| `kali-tools-forensics`             | Volatility, Autopsy, Binwalk...     |
| `kali-tools-passwords`             | Hashcat, John, Hydra...             |
| `kali-tools-information-gathering` | Nmap, Maltego, theHarvester...      |
| `kali-tools-sniffing-spoofing`     | Wireshark, Ettercap, Bettercap...   |
| `kali-tools-post-exploitation`     | Post-explot, pivoting...            |

#### Instalación de metapaquetes esenciales

```bash
sudo apt install kali-tools-web
sudo apt install kali-tools-information-gathering
sudo apt install kali-tools-top10
sudo apt install kali-linux-default
```

### Herramientas esenciales por categoría

#### Reconocimiento y OSINT

* `nmap` — escaneo de red y detección de servicios
* `theHarvester` — OSINT de emails, dominios e IPs
* `amass` — enumeración de subdominios
* `dnsrecon` / `dnsenum` — enumeración DNS
* `maltego` — OSINT visual (requiere cuenta)
* `recon-ng` — framework de reconocimiento modular

#### Fuzzing y enumeración web

* `gobuster` — fuzzing de directorios y subdominios
* `ffuf` — fuzzing HTTP muy rápido y flexible
* `feroxbuster` — fuzzing recursivo
* `nikto` — escáner de vulnerabilidades web
* `wpscan` — específico para WordPress

#### Explotación

* `metasploit-framework` — el framework de explotación más completo
* `searchsploit` — buscar en Exploit-DB localmente
* `sqlmap` — SQL Injection automatizado

#### Contraseñas y hashes

* `hashcat` — cracking con GPU (el más rápido)
* `john` — John the Ripper, muy versátil
* `hydra` — bruteforce de servicios online

Los modos más usados de hashcat:

| Modo (`-m`) | Tipo de hash                    |
| ----------- | ------------------------------- |
| `0`         | MD5                             |
| `1000`      | NTLM (Windows)                  |
| `1800`      | SHA512crypt (Linux /etc/shadow) |
| `22000`     | WPA2                            |
| `5600`      | NetNTLMv2 (MSCHAPv2)            |

#### Hacking Wi-Fi

* `aircrack-ng` suite — captura y cracking de WPA2
* `hcxtools` / `hcxdumptool` — PMKID attack (sin necesidad de clientes)
* `reaver` — ataque WPS (Pixie Dust y fuerza bruta)

#### Post-explotación y escalada

* `linpeas.sh` — escalada de privilegios Linux automatizada
* `winpeas.exe` — escalada de privilegios Windows
* `pwncat-cs` — shell reversa mejorada

### Wordlists

Kali incluye wordlists preinstaladas en `/usr/share/wordlists/`. La más famosa es **rockyou.txt**, con más de 14 millones de contraseñas reales de brechas de datos.

```bash
# Descomprimir rockyou.txt (viene comprimida por defecto)
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Para tener la colección más completa, instalar **SecLists**:

```bash
sudo apt install seclists
```

| Ruta en SecLists         | Contenido                            |
| ------------------------ | ------------------------------------ |
| `Discovery/Web-Content/` | Directorios y archivos web           |
| `Discovery/DNS/`         | Subdominios                          |
| `Passwords/`             | Contraseñas y wordlists              |
| `Usernames/`             | Nombres de usuario                   |
| `Fuzzing/`               | Payloads para fuzzing (SQLi, XSS...) |
