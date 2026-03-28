---
icon: linux
---

# Arch Linux

Arch Linux es una distribución rolling-release y minimalista que sigue el principio KISS (_Keep It Simple, Stupid_). No instala nada que no hayas pedido explícitamente: el sistema base es kernel + shell, y desde ahí construyes exactamente lo que necesitas.

### Datos clave

| Campo                      | Valor                                    |
| -------------------------- | ---------------------------------------- |
| **Base de paquetes**       | tar.zst (formato propio)                 |
| **Init system**            | systemd                                  |
| **Escritorio por defecto** | Ninguno (lo instalas tú)                 |
| **Ciclo de versiones**     | Rolling release (sin versiones)          |
| **Soporte**                | Indefinido mientras actualices           |
| **Empresa**                | Comunidad (sin empresa)                  |
| **Documentación**          | [Arch Wiki](https://wiki.archlinux.org/) |

#### ¿Qué es rolling release?

En Arch no existen versiones. El sistema siempre está en la última versión de todo: kernel, GNOME, toolchain... Un update de hoy da el software más reciente disponible. La ventaja es que nunca tienes software obsoleto, la desventaja es que actualizaciones muy grandes a veces requieren leer las notas de versión antes de ejecutarlas.

### Gestor de paquetes — Pacman

Pacman es extremadamente rápido y usa una sintaxis de flags de una sola letra. La lógica de los flags es consistente: `-S` (sync, repositorios remotos), `-Q` (query, paquetes instalados), `-R` (remove, eliminar).

#### Operaciones esenciales

```bash
# Actualizar el sistema COMPLETO (siempre hacer esto antes de instalar)
sudo pacman -Syu

# Buscar paquetes en repositorios
pacman -Ss nmap

# Ver información de un paquete
pacman -Si nmap    # en repositorios
pacman -Qi nmap    # instalado localmente

# Instalar y eliminar
sudo pacman -S nmap wireshark git
sudo pacman -Rns nmap    # elimina paquete + dependencias + archivos de config
```

#### Consultas con pacman

Pacman tiene un sistema de consultas muy potente para explorar el sistema:

| Comando                    | Descripción                                                                |
| -------------------------- | -------------------------------------------------------------------------- |
| `pacman -Q`                | Listar todos los paquetes instalados                                       |
| `pacman -Qe`               | Solo los instalados explícitamente (no como dependencias)                  |
| `pacman -Qdt`              | Paquetes huérfanos (dependencias ya no necesarias)                         |
| `pacman -Ql nmap`          | Archivos que instaló un paquete                                            |
| `pacman -Qo /usr/bin/nmap` | A qué paquete pertenece un archivo                                         |
| `pacman -Qm`               | Paquetes no oficiales (AUR o instalados manualmente)                       |
| `pacman -F nmap`           | Buscar un archivo entre todos los paquetes (requiere `pacman -Fy` primero) |

#### Limpiar caché

Pacman guarda todas las versiones descargadas en `/var/cache/pacman/pkg/`. Es bueno limpiar periódicamente:

```bash
sudo pacman -Sc    # elimina versiones antiguas, guarda las instaladas
sudo pacman -Scc   # elimina todo el caché

# Con paccache (del paquete pacman-contrib) — más preciso:
sudo pacman -S pacman-contrib
sudo paccache -r       # guarda las 3 últimas versiones de cada paquete
sudo paccache -rk 1    # guarda solo 1 versión
```

### AUR — Arch User Repository

El AUR es el repositorio comunitario de Arch. Contiene miles de paquetes mantenidos por usuarios — prácticamente cualquier software disponible para Linux tiene un PKGBUILD en el AUR. Es una de las mayores ventajas de Arch frente a otras distros.

#### ¿Cómo funciona?

El AUR no se gestiona directamente con pacman. Cada paquete del AUR es un repositorio Git que contiene un **PKGBUILD**: un script bash que define de dónde descargar el código fuente, cómo compilarlo y cómo instalarlo. La herramienta `makepkg` lee ese PKGBUILD y construye el paquete.

#### Proceso manual (sin AUR helper)

```bash
git clone https://aur.archlinux.org/nombre-paquete.git
cd nombre-paquete
less PKGBUILD          # SIEMPRE leer el PKGBUILD antes de instalar
makepkg -si            # compilar e instalar (resuelve dependencias automáticamente)
```

> Lee siempre el PKGBUILD antes de ejecutar `makepkg`. Es código que se ejecuta en tu sistema. Verificar que descarga el código fuente de sitios legítimos y que no ejecuta nada sospechoso.

#### AUR Helpers — yay y paru

Los AUR helpers automatizan el proceso de búsqueda, descarga y construcción desde el AUR. Los más usados son **yay** y **paru**.

**yay** usa exactamente la misma sintaxis que pacman más soporte del AUR:

```bash
# Instalar yay (proceso manual, solo la primera vez)
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si

# Uso diario
yay -Syu              # actualizar sistema + AUR
yay -S nombre-paquete # busca en repos oficiales y AUR
yay -Ss término       # buscar en todo
yay -Yc               # limpiar dependencias no usadas del AUR
```

**paru** es la alternativa moderna escrita en Rust. Muestra el PKGBUILD por defecto antes de instalar, lo que lo hace más seguro:

```bash
git clone https://aur.archlinux.org/paru.git
cd paru && makepkg -si

paru                   # sin argumentos = full system upgrade
paru -S nombre-paquete
```

### Repositorios

Los repositorios oficiales de Arch se configuran en `/etc/pacman.conf`:

| Repositorio  | Contenido                                 |
| ------------ | ----------------------------------------- |
| `[core]`     | Paquetes esenciales del sistema           |
| `[extra]`    | Paquetes adicionales oficiales            |
| `[multilib]` | Paquetes de 32 bits (para Steam, Wine...) |

Para habilitar `multilib`, descomentar estas líneas en `/etc/pacman.conf` y luego ejecutar `sudo pacman -Sy`:

```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

#### Optimizar mirrors con Reflector

```bash
sudo pacman -S reflector
sudo reflector --country Spain,France,Germany \
    --protocol https --sort rate \
    --save /etc/pacman.d/mirrorlist
```

### Mantenimiento del sistema

Arch requiere algo más de atención que otras distros. Buenas prácticas:

Leer las noticias antes de updates grandes. Visitar [archlinux.org/news](https://archlinux.org/news) o instalar `informant` del AUR, que muestra las noticias antes de cada update automáticamente.

**Eliminar paquetes huérfanos regularmente:**

```bash
sudo pacman -Rns $(pacman -Qtdq) 2>/dev/null
```

**Gestionar archivos `.pacnew`.** Cuando un paquete actualizado trae cambios en sus archivos de configuración, Arch los guarda como `.pacnew` en lugar de sobreescribir tu configuración. Revisarlos periódicamente:

```bash
find / -name "*.pacnew" 2>/dev/null
sudo pacdiff    # herramienta interactiva para gestionarlos
```

**Downgrade si algo se rompe:**

```bash
# Desde el caché local
sudo pacman -U /var/cache/pacman/pkg/nombre-paquete-version.tar.zst

# Con la herramienta downgrade del AUR
paru -S downgrade
sudo downgrade nombre-paquete    # menú interactivo con versiones disponibles
```

### La Arch Wiki

La **Arch Wiki** es tan completa que incluso usuarios de otras distribuciones la consultan habitualmente. Antes de buscar en Google cualquier problema de Linux, buscar primero en [wiki.archlinux.org](https://wiki.archlinux.org/) suele dar la explicación más técnica y precisa disponible. Es la mejor documentación de Linux que existe.

### Distribuciones famosas basadas en Arch

| Distro          | Descripción                                                 |
| --------------- | ----------------------------------------------------------- |
| **EndeavourOS** | Arch con instalador simple, muy fiel al original            |
| **Manjaro**     | Arch con instalador gráfico y herramientas adicionales      |
| **Garuda**      | Arch con BTRFS y snapshots automáticos                      |
| **CachyOS**     | Arch optimizado para rendimiento con kernels personalizados |

### Cheatsheet de arch

| Acción                                 | Comando                                                 |
| -------------------------------------- | ------------------------------------------------------- |
| Actualizar sistema completo            | `sudo pacman -Syu`                                      |
| Ver versión del kernel                 | `uname -r`                                              |
| Ver paquetes instalados explícitamente | `pacman -Qe`                                            |
| Ver paquetes huérfanos                 | `pacman -Qdt`                                           |
| Eliminar huérfanos                     | `sudo pacman -Rns $(pacman -Qtdq)`                      |
| Ver log de pacman                      | `cat /var/log/pacman.log`                               |
| Crear usuario con sudo                 | `sudo useradd -m -G wheel usuario`                      |
| Habilitar sudo para wheel              | Editar `visudo`, descomentar `%wheel ALL=(ALL:ALL) ALL` |
