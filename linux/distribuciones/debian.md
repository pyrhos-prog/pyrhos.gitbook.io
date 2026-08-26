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

# Debian

Debian es una de las distribuciones más antiguas y respetadas de Linux, conocida por su estabilidad y su compromiso con el software libre. Es la base de decenas de distribuciones: Ubuntu, Kali, Raspberry Pi OS, Linux Mint...

### Datos clave

| Campo                      | Valor                        |
| -------------------------- | ---------------------------- |
| **Base de paquetes**       | DEB                          |
| **Init system**            | systemd (desde Debian 8)     |
| **Escritorio por defecto** | GNOME (también KDE, XFCE...) |
| **Ciclo de versiones**     | \~2 años (sin fecha fija)    |
| **Soporte por versión**    | \~5 años (3 activo + 2 LTS)  |
| **Empresa**                | Comunidad (Debian Project)   |
| **Versión actual**         | Debian 13 "Trixie"           |

#### Ramas de Debian

Debian tiene tres ramas activas con distintos niveles de estabilidad:

| Rama         | Nombre   | Descripción                                                    |
| ------------ | -------- | -------------------------------------------------------------- |
| **stable**   | bookworm | Versiones congeladas, muy estable. Ideal para servidores.      |
| **testing**  | trixie   | Próxima versión en desarrollo. Más actualizada, menos estable. |
| **unstable** | sid      | Siempre con las últimas versiones. Puede tener bugs.           |

Existe además **backports**: un repositorio que porta paquetes recientes a _stable_, permitiendo tener software más nuevo sin perder la estabilidad general del sistema.

### Gestor de paquetes — APT

APT (Advanced Package Tool) es el gestor de paquetes de Debian y todos sus derivados. Hay tres herramientas relacionadas con niveles distintos de abstracción:

| Herramienta | Uso recomendado                                      |
| ----------- | ---------------------------------------------------- |
| `apt`       | Interfaz moderna para el usuario (desde Debian 8)    |
| `apt-get`   | Más opciones, más predecible en scripts              |
| `dpkg`      | Bajo nivel, trabaja directamente con archivos `.deb` |

#### Operaciones esenciales

El flujo básico comienza siempre con `sudo apt update` para actualizar el índice de repositorios, y luego `sudo apt upgrade` para actualizar los paquetes. Para una actualización completa que también resuelve cambios de dependencias: `sudo apt full-upgrade`.

Para instalar: `sudo apt install nmap`. Para eliminar, hay dos variantes: `sudo apt remove nmap` elimina el paquete pero deja los archivos de configuración, mientras que `sudo apt purge nmap` elimina todo incluyendo la configuración. `sudo apt autoremove` limpia dependencias que quedaron huérfanas.

```bash
# Ciclo completo de actualización
sudo apt update && sudo apt full-upgrade

# Instalar y eliminar
sudo apt install nmap wireshark git
sudo apt remove nmap          # elimina el paquete, conserva config
sudo apt purge nmap           # elimina paquete y config
sudo apt autoremove --purge   # limpia huérfanos y sus configs

# Buscar e informar
apt search nmap
apt show nmap

# Limpiar caché
sudo apt clean       # elimina todos los .deb cacheados
sudo apt autoclean   # elimina solo los obsoletos
```

#### Gestión directa con dpkg

Cuando se descarga un `.deb` manualmente, `sudo dpkg -i paquete.deb` lo instala. Si hay dependencias sin resolver, `sudo apt install -f` las arregla. Comandos útiles de dpkg:

* `dpkg -L nmap` — listar todos los archivos que instaló un paquete
* `dpkg -S /usr/bin/nmap` — ver a qué paquete pertenece un archivo del sistema
* `dpkg -s nmap` — información del paquete instalado
* `dpkg --list` — listar todos los paquetes instalados

### Repositorios — sources.list

Los repositorios se configuran en `/etc/apt/sources.list` y en archivos dentro de `/etc/apt/sources.list.d/`. El formato de cada línea es `deb URL RELEASE COMPONENTS`.

Los **componentes** definen qué tipo de software se incluye:

| Componente          | Contenido                                  |
| ------------------- | ------------------------------------------ |
| `main`              | Software libre (DFSG-compatible)           |
| `contrib`           | Software libre con dependencias privativas |
| `non-free`          | Software privativo                         |
| `non-free-firmware` | Firmware privativo (añadido en Debian 12)  |

#### sources.list típico para Debian 12

```
deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```

#### Usar backports

Para añadir backports y tener versiones más recientes de paquetes específicos en _stable_:

```bash
# Añadir al sources.list:
# deb http://deb.debian.org/debian bookworm-backports main contrib non-free non-free-firmware

# Instalar desde backports:
sudo apt install -t bookworm-backports nombre-paquete
```

#### Añadir repositorios de terceros

El método moderno usa `/etc/apt/sources.list.d/` con claves GPG separadas:

```bash
# Ejemplo: añadir repositorio de Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/debian $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update && sudo apt install docker-ce
```

### Firewall

Debian no instala ningún firewall por defecto. Las opciones más habituales:

**UFW** (el más simple para empezar):

```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw status verbose
```

**nftables** (el backend moderno, sustituyó a iptables):

```bash
sudo apt install nftables
sudo systemctl enable nftables
sudo nft list ruleset
```

### AppArmor

Debian usa **AppArmor** como sistema de control de acceso, a diferencia de Fedora/RHEL que usan SELinux.

| Modo       | Comportamiento            |
| ---------- | ------------------------- |
| `enforce`  | Activo y bloqueando       |
| `complain` | Solo registra, no bloquea |
| `disabled` | Desactivado               |

```bash
sudo aa-status                    # ver estado y perfiles cargados
sudo aa-complain /usr/sbin/nginx  # poner perfil en modo complain
sudo aa-enforce /usr/sbin/nginx   # volver a enforce
sudo journalctl -k | grep apparmor  # ver logs de AppArmor
```

### Cheatsheet

| Acción                              | Comando                             |
| ----------------------------------- | ----------------------------------- |
| Ver versión de Debian               | `cat /etc/debian_version`           |
| Ver info completa del sistema       | `lsb_release -a`                    |
| Ver arquitectura                    | `dpkg --print-architecture`         |
| Añadir usuario a sudo               | `sudo usermod -aG sudo usuario`     |
| Reinstalar paquete sin tocar config | `sudo apt install --reinstall nmap` |
| Simular instalación (dry-run)       | `sudo apt install -s nmap`          |
| Ver dependencias de un paquete      | `apt-cache depends nmap`            |
| Ver qué depende de un paquete       | `apt-cache rdepends nmap`           |
| Descargar paquete sin instalar      | `apt download nmap`                 |

### Distribuciones Debian Based

<div data-full-width="false" data-with-frame="true"><figure><img src="https://36118879-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F5YP0HIeDL3z0KLMojf1y%2Fuploads%2Fl0KMLZhoTBwWWePjVqOB%2Fimage.png?alt=media&#x26;token=48b45d21-11d4-4a9c-a70e-90f6f0485aa8" alt="" width="375"><figcaption></figcaption></figure></div>
