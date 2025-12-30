# ldapdomaindump

{% stepper %}
{% step %}
### 1. Información general del sistema

Reúne datos básicos que describen la máquina y su estado actual.

{% tabs %}
{% tab title="Linux" %}
* Hostname
  * comando: `hostnamectl` / `cat /etc/hostname`
* Fecha y hora
  * comando: `date` / `timedatectl`
* Kernel y arquitectura
  * comando: `uname -a` / `uname -m`
* Distribución y versión
  * comando: `cat /etc/os-release`
* Uptime y carga
  * comando: `uptime` / `top -b -n1 | head -n5`
* Variables de entorno relevantes
  * comando: `env` / `printenv`
{% endtab %}

{% tab title="Windows" %}
* Nombre del equipo y dominio
  * comando (PowerShell): `Get-ComputerInfo | Select CsName, WindowsProductName, WindowsVersion`
* Fecha y hora
  * comando: `Get-Date`
* Versión del SO y arquitectura
  * comando: `systeminfo` o `Get-WmiObject -Class Win32_OperatingSystem`
* Uptime
  * comando: `(Get-CimInstance -ClassName Win32_OperatingSystem).LastBootUpTime`
* Variables de entorno
  * comando: `Get-ChildItem Env:`
{% endtab %}

{% tab title="macOS" %}
* Hostname
  * comando: `scutil --get ComputerName` / `hostname`
* Versión de macOS
  * comando: `sw_vers`
* Kernel y arquitectura
  * comando: `uname -a`
* Fecha y uptime
  * comando: `date` / `uptime`
* Variables de entorno
  * comando: `printenv`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 2. Hardware y recursos

Identifica CPU, memoria, dispositivos y aceleradores.

{% tabs %}
{% tab title="Linux" %}
* CPU: `lscpu` o `cat /proc/cpuinfo`
* Memoria: `free -h` / `cat /proc/meminfo`
* Discos y particiones: `lsblk -f` / `fdisk -l` / `blkid`
* Información SMART (si aplica): `smartctl -a /dev/sdX`
* PCI/USB devices: `lspci -v` / `lsusb`
* GPU: `lspci | grep -i vga` / `nvidia-smi` (si existe)
{% endtab %}

{% tab title="Windows" %}
* CPU y memoria: `Get-WmiObject -Class Win32_Processor` / `Get-WmiObject -Class Win32_ComputerSystem`
* Discos y particiones: `Get-PhysicalDisk` / `Get-Partition` / `Get-Volume`
* Información SMART: herramientas como `Get-PhysicalDisk` o utilidades del fabricante
* Dispositivos PCI/USB: `Get-PnpDevice`
* GPU: `Get-WmiObject Win32_VideoController`
{% endtab %}

{% tab title="macOS" %}
* CPU/memoria: `sysctl -n machdep.cpu.brand_string` / `sysctl hw.memsize`
* Discos y particiones: `diskutil list` / `df -h`
* PCI/USB: `system_profiler SPUSBDataType` / `system_profiler SPPCIDataType`
* GPU: `system_profiler SPDisplaysDataType`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 3. Red y conectividad

Extrae interfaces, rutas, conexiones activas y resolución de nombres.

{% tabs %}
{% tab title="Linux" %}
* Interfaces y direcciones IP: `ip addr show` / `ifconfig -a`
* Rutas y gateway: `ip route` / `route -n`
* DNS configurado: `cat /etc/resolv.conf`
* Conexiones activas y sockets: `ss -tunap` / `netstat -tunap`
* Tablas ARP: `ip neigh`
* Reglas de firewall: `iptables -L -n -v` / `nft list ruleset`
* Conexion saliente: `curl -I http://example.com` o `ss -tn sport = :80`
{% endtab %}

{% tab title="Windows" %}
* Interfaces y IP: `ipconfig /all` / `Get-NetIPAddress`
* Rutas: `route print` / `Get-NetRoute`
* DNS: `Get-DnsClientServerAddress`
* Conexiones activas: `Get-NetTCPConnection` / `netstat -ano`
* Tabla ARP: `arp -a`
* Firewall: `Get-NetFirewallRule` / `netsh advfirewall firewall show rule name=all`
{% endtab %}

{% tab title="macOS" %}
* Interfaces y IP: `ifconfig` / `ipconfig getifaddr en0`
* Rutas: `netstat -nr` / `route get default`
* DNS: `scutil --dns`
* Conexiones activas: `lsof -i` / `netstat -an`
* Firewall: `sudo /usr/libexec/ApplicationFirewall/socketfilterfw --listapps`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 4. Usuarios, grupos y sesiones

Lista usuarios locales, sesiones activas y privilegios.

{% tabs %}
{% tab title="Linux" %}
* Usuarios locales: `cat /etc/passwd`
* Grupos: `getent group`
* Últimos inicios de sesión: `last` / `lastlog`
* Usuarios actualmente conectados: `who` / `w`
* UID 0 accounts: `awk -F: '($3 == 0){print}' /etc/passwd`
* Sudoers: `cat /etc/sudoers` y archivos en `/etc/sudoers.d/`
{% endtab %}

{% tab title="Windows" %}
* Cuentas locales: `Get-LocalUser` (PowerShell) / `net user`
* Grupos: `Get-LocalGroup` / `net localgroup`
* Sesiones RDP/usuarios conectados: `quser` / `query user`
* Administradores: `Get-LocalGroupMember -Group Administrators`
* Últimos inicios/ eventos: revisar Event Viewer (Security log)
{% endtab %}

{% tab title="macOS" %}
* Usuarios: `dscl . -list /Users` / `dscl . -read /Users/username`
* Grupos: `dscl . -list /Groups`
* Usuarios conectados: `who` / `last`
* Administradores: `dscl . -read /Groups/admin GroupMembership`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 5. Procesos, servicios y tareas programadas

Investiga procesos en ejecución, servicios persistentes y tareas automáticas.

{% tabs %}
{% tab title="Linux" %}
* Procesos: `ps aux --sort=-%mem | head` / `top` / `htop`
* Servicios systemd: `systemctl list-units --type=service --state=running`
* Cron jobs: `crontab -l` (por usuario) y revisar `/etc/crontab` y `/etc/cron.*`
* System startup scripts: `/etc/init.d/`, `/etc/rc.local`
* Tareas en at/Anacron: `atq` / `ls /etc/cron*`
{% endtab %}

{% tab title="Windows" %}
* Procesos: `Get-Process` / `tasklist`
* Servicios: `Get-Service` / `sc query`
* Tareas programadas: `Get-ScheduledTask` / `schtasks /query /fo LIST /v`
* Inicio automático (Run keys, services): revisar registro en `HKLM\...\Run` y `HKCU\...\Run`
{% endtab %}

{% tab title="macOS" %}
* Procesos: `ps aux` / `top`
* LaunchDaemons/Agents: `/Library/LaunchDaemons`, `/Library/LaunchAgents`, `~/Library/LaunchAgents`
* Tareas programadas: `launchctl list`
* Cron (si existe): `crontab -l`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 6. Almacenamiento y artefactos en disco

Localiza archivos importantes, puntos de montaje y uso de espacio.

{% tabs %}
{% tab title="Linux" %}
* Uso de disco: `df -h` / `du -sh /var/* | sort -hr | head`
* Montajes: `mount` / `lsblk`
* Búsqueda de archivos recientes: `find / -type f -mtime -7 2>/dev/null | head`
* Archivos de configuración: `/etc/` (revisa modificaciones)
* Binaries sospechosos en PATH: `which <cmd>` / `whereis`
* Listado de paquetes instalados: `dpkg -l` (Debian) / `rpm -qa` (RedHat) / `pacman -Q`
{% endtab %}

{% tab title="Windows" %}
* Uso de disco: `Get-PSDrive -PSProvider FileSystem` / `fsutil volume diskfree C:`
* Archivos recientes: `Get-ChildItem -Path C:\ -Recurse | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}`
* Programas instalados: `Get-WmiObject -Class Win32_Product` (o revisar registro)
* Registros de configuración y ubicaciones de perfil: `C:\Users\<user>\AppData\*`
{% endtab %}

{% tab title="macOS" %}
* Uso de disco: `df -h` / `du -sh /Users/*`
* Montajes y APFS: `diskutil list` / `mount`
* Archivos recientes: `mdfind "kMDItemFSContentChangeDate > $time"` o `find`
* Aplicaciones instaladas: `/Applications` y `pkgutil --pkgs`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 7. Logs y auditoría

Recopila logs del sistema, de seguridad y aplicaciones relevantes.

{% tabs %}
{% tab title="Linux" %}
* Systemd journal: `journalctl -xe --no-pager`
* Logs tradicionales: `/var/log/syslog` o `/var/log/messages`, `/var/log/auth.log`
* Logs de aplicaciones: revisa `/var/log/<app>/`
* Logs de acceso SSH: `/var/log/auth.log` o `journalctl -u ssh`
* Filtrado por tiempo: `journalctl --since "2025-01-01" --until "2025-01-02"`
{% endtab %}

{% tab title="Windows" %}
* Event Viewer: Security, System, Application logs
  * PowerShell: `Get-WinEvent -LogName Security -MaxEvents 100`
* IIS logs: `C:\inetpub\logs\LogFiles\`
* PowerShell transcription / logs: revisar políticas y ubicaciones configuradas
{% endtab %}

{% tab title="macOS" %}
* Unified logging: `log show --predicate 'process == "sshd"' --last 1d`
* Syslog locations: `/var/log/` (varios archivos)
* Console app para revisar eventos en GUI
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 8. Seguridad y configuraciones de hardening

Revisa controles de seguridad, privilegios y auditoría.

{% tabs %}
{% tab title="Linux" %}
* Permisos y SUID/SGID binarios: `find / -perm /6000 -type f -exec ls -l {} \; 2>/dev/null`
* Módulos de kernel cargados: `lsmod`
* SELinux/AppArmor estado: `sestatus` / `aa-status`
* Usuarios con sudo: `grep '^sudo' /etc/group` / `getent group sudo`
* Reglas de firewall: `iptables -L` / `nft list ruleset`
* SSH config: `cat /etc/ssh/sshd_config`
{% endtab %}

{% tab title="Windows" %}
* Políticas de auditoría y GPO: `Get-GPOReport`
* UAC: estado en registro/PowerShell
* Cuentas con privilegios: miembros del grupo Administrators
* Firewall y reglas: `Get-NetFirewallRule`
* Antivirus/EDR: comprobar servicios y procesos conocidos
{% endtab %}

{% tab title="macOS" %}
* SIP status: `csrutil status` (requiere recuperación para cambiar)
* Gatekeeper/App Store settings
* Usuarios admin y sudoers
* Firewall y políticas de seguridad: `defaults read /Library/Preferences/com.apple.alf`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 9. Persistencia, artefactos de compromiso y evidencia forense

Busca mecanismos comunes de persistencia y señales de compromiso.

* Entradas en arranque y autoarranque (Run keys, RunOnce, init scripts, systemd services, launchd).
* Cron/Task Scheduler maliciosos.
* Binaries desconocidos o con timestamps recientes en binarios del sistema.
* Conexiones persistentes salientes (reverse shells, túneles).
* Módulos/driver sospechosos cargados.
* Historial de comandos (bash\_history, PowerShell history, .zsh\_history) — ten en cuenta que se puede manipular.
* Revisar credentials almacenadas (ssh keys en \~/.ssh, archivos .pem/.ppk, credenciales de navegador) con cautela y conforme a las políticas legales.
* Artefactos de ofuscación o empaquetado (packed binaries, scripts en temp).
{% endstep %}

{% step %}
### 10. Herramientas útiles y comandos rápidos

Lista breve de utilidades para análisis in situ y recolección.

{% tabs %}
{% tab title="Linux" %}
* Sysinternals-like: `ps`, `ss`, `lsof`, `strace`, `tcpdump`, `nmap` (si está), `chkconfig`/`systemctl`
* Recolección rápida: `tar -czf /tmp/host-collect.tgz /etc /var/log /home /root` (ajustar según políticas)
* Hashing: `sha256sum <file>`
{% endtab %}

{% tab title="Windows" %}
* Herramientas integradas: `tasklist`, `netstat`, `wevtutil`, `powershell` (Get-Process, Get-Service)
* Sysinternals suite: Procmon, Autoruns, PsExec, ProcDump
* Recolección: PowerShell script para exportar event logs y listado de servicios/usuarios
{% endtab %}

{% tab title="macOS" %}
* Utilidades: `lsof`, `netstat`, `dtrace` (con privilegios), `system_profiler`
* Recolección: empaquetar `/var/log`, `/Library/Preferences`, `/Users/<user>/Library/Logs`
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### 11. Buenas prácticas y precauciones

{% hint style="warning" %}
* Actúa siempre conforme a la ley y políticas internas. No accedas, modifiques ni extraigas datos sin autorización.
* Evita comandos destructivos (p. ej. `rm -rf /`) y no uses herramientas que puedan alterar evidencia si necesitas preservarla.
* Preferir recolección de evidencia en modo lectura cuando sea posible (mount -o ro, exports forensics snapshots).
{% endhint %}

{% hint style="info" %}
* Documenta todo lo que ejecutes (comandos y salidas) y salva metadatos (timestamps, hashes).
* Si eres parte de respuesta a incidentes, coordina con el equipo forense para captura segura.
{% endhint %}
{% endstep %}
{% endstepper %}
