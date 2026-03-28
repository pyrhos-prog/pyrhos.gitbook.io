---
icon: fedora
---

# Fedora

Fedora es una distribución patrocinada por Red Hat con filosofía _bleeding-edge_: siempre lleva las versiones más recientes del kernel, GNOME y el toolchain. Es la distribución upstream de RHEL — lo que entra en Fedora hoy llega a los servidores enterprise en unos años.

### Datos clave

| Campo                      | Valor                  |
| -------------------------- | ---------------------- |
| **Base de paquetes**       | RPM                    |
| **Init system**            | systemd                |
| **Escritorio por defecto** | GNOME (última versión) |
| **Ciclo de versiones**     | \~6 meses              |
| **Soporte por versión**    | \~13 meses             |
| **Empresa**                | Red Hat / IBM          |

### Gestor de paquetes — DNF

DNF (Dandified YUM) es el gestor de paquetes de Fedora desde la versión 22. Toda la gestión de software pasa por él.

#### Operaciones esenciales

Para actualizar el sistema completo se usa `sudo dnf update` . Para buscar un paquete sin instalarlo, `dnf search <paquete>` devuelve los resultados de los repositorios activos, y `dnf info <paquete>` muestra descripción, versión y dependencias antes de instalar.

Para instalar se usa `sudo dnf install <paquete>`, pudiendo encadenar varios paquetes. Para eliminar, `sudo dnf remove <paquete>`, y `sudo dnf autoremove` limpia las dependencias que quedaron huérfanas.

```bash
# Actualizar el sistema
sudo dnf update

# Instalar y eliminar
sudo dnf install nmap wireshark git
sudo dnf remove nmap
sudo dnf autoremove

# Buscar e informar
dnf search nmap
dnf info nmap

# Limpiar caché
sudo dnf clean all
```

#### Historial y rollback

Una de las ventajas de DNF es el **historial de transacciones**. Con `dnf history` se ve una lista numerada de todas las operaciones realizadas. Si una actualización rompe algo, `sudo dnf history undo N` deshace esa transacción completa.

```bash
dnf history               # ver historial numerado
dnf history info 5        # detalle de la transacción 5
sudo dnf history undo 5   # deshacer la transacción 5
```

#### Repositorios y RPM Fusion

Por defecto Fedora incluye solo software libre. Para instalar software privativo (drivers NVIDIA, códecs multimedia...) hay que añadir **RPM Fusion**:

```bash
sudo dnf install \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

Para ver repositorios activos: `dnf repolist`. Para repositorios comunitarios de proyectos individuales existe **COPR**: `sudo dnf copr enable usuario/repo`.

#### Actualizar de versión (F40 - F41)

```bash
sudo dnf install dnf-plugin-system-upgrade
sudo dnf system-upgrade download --releasever=41
sudo dnf system-upgrade reboot
```

### Gestión directa con RPM

Cuando se descarga un `.rpm` manualmente, conviene instalarlo con `sudo dnf install paquete.rpm` en lugar de `rpm -ivh` porque DNF resuelve automáticamente las dependencias. Algunos comandos útiles de `rpm` directamente:

* `rpm -ql nmap` — listar todos los archivos que instaló un paquete
* `rpm -qf /usr/bin/nmap` — ver a qué paquete pertenece un archivo del sistema
* `rpm -qi nmap` — información del paquete instalado
* `rpm -V nmap` — verificar integridad de los archivos del paquete

### Flatpak

Fedora integra Flatpak de serie como alternativa para aplicaciones de escritorio. Primero hay que añadir Flathub como fuente:

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install flathub org.mozilla.firefox
flatpak update
flatpak uninstall --unused    # limpiar runtimes no usados
```

### Systemd

Fedora usa systemd para la gestión de servicios. Una convención específica: el servicio SSH aquí se llama `sshd` (en Debian se llama `ssh`).

```bash
sudo systemctl start sshd
sudo systemctl enable --now sshd    # habilitar Y arrancar a la vez
sudo systemctl status sshd

# Logs con journalctl
journalctl -u sshd -f               # en tiempo real (follow)
journalctl -b                       # desde el último arranque
journalctl --since "1 hour ago"
```

### Firewall — firewalld

Fedora usa **firewalld** con el concepto de _zonas_ en lugar de gestionar iptables directamente.

```bash
sudo firewall-cmd --list-all                                         # ver reglas actuales
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent      # abrir puerto
sudo firewall-cmd --zone=public --add-service=https --permanent      # abrir servicio
sudo firewall-cmd --reload                                           # aplicar permanentes
sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent   # cerrar puerto
```

> Sin el flag `--permanent` las reglas desaparecen en el siguiente reboot. Siempre añadir `--permanent` seguido de `--reload` para que los cambios persistan.

### SELinux

Fedora viene con SELinux en modo **enforcing** por defecto. Es el sistema de control de acceso obligatorio del ecosistema Red Hat, una capa de seguridad que muchas distros desactivan pero que aquí se mantiene activa.

| Modo         | Comportamiento                                |
| ------------ | --------------------------------------------- |
| `enforcing`  | Aplica las políticas, bloquea lo no permitido |
| `permissive` | Solo registra, no bloquea (útil para debug)   |
| `disabled`   | Completamente desactivado (requiere reboot)   |

Para ver el modo actual: `getenforce`. Para cambiar temporalmente a permissive: `sudo setenforce 0`. El cambio permanente va en `/etc/selinux/config`.

Cuando SELinux bloquea algo, `sudo sealert -a /var/log/audit/audit.log` explica exactamente qué bloqueó y proporciona el comando correcto para resolverlo sin necesidad de deshabilitar SELinux.

```bash
getenforce                                         # ver modo actual
sudo setenforce 0                                  # cambio temporal a permissive
sudo setsebool -P httpd_can_network_connect on     # activar booleano permanente
ls -Z /var/www/html/                               # ver contexto SELinux de archivos
```

> Antes de deshabilitar SELinux ante un problema, ejecutar `sealert` para entender qué está bloqueando. Desactivarlo es la solución fácil pero elimina una capa de seguridad importante.

### Variantes de Fedora

| Variante        | Descripción                                |
| --------------- | ------------------------------------------ |
| **Workstation** | GNOME, escritorio principal                |
| **Server**      | Sin escritorio, para servidores            |
| **Silverblue**  | Sistema inmutable con rpm-ostree y Flatpak |
| **Kinoite**     | Como Silverblue pero con KDE               |
| **Spins**       | KDE, XFCE, i3, MATE...                     |
| **CoreOS**      | Para contenedores y clusters Kubernetes    |

### Cheatsheet de Fedora

| Acción                          | Comando                          |
| ------------------------------- | -------------------------------- |
| Ver versión de Fedora           | `cat /etc/fedora-release`        |
| Ver versión del kernel          | `uname -r`                       |
| Añadir usuario a sudoers        | `sudo usermod -aG wheel usuario` |
| Ver interfaces de red           | `ip a`                           |
| Ver conexiones (NetworkManager) | `nmcli device status`            |
| Ver uso de disco                | `df -h`                          |
| Ver RAM                         | `free -h`                        |
| Ver hardware                    | `lscpu` / `lspci` / `lsusb`      |

> Las habilidades con DNF, systemd y SELinux son directamente transferibles a RHEL y CentOS, que dominan el mercado de servidores enterprise. Fedora es la mejor forma de aprender ese ecosistema sin pagar licencias de Red Hat.

