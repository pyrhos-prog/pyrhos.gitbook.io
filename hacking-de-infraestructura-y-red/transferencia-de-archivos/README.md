---
icon: building-magnifying-glass
---

# Transferencia de archivos

Durante un pentest, la capacidad de transferir archivos hacia y desde un sistema comprometido es una habilidad crítica. Las herramientas de enumeración, los exploits compilados, los scripts de post-explotación y el loot extraído necesitan moverse entre sistemas — y los defensores, firewalls, proxies y políticas de aplicaciones lo dificultan activamente.

### Por qué importa dominar múltiples métodos

Ningún método funciona en todos los entornos. El siguiente escenario ilustra el problema:

Durante un engagement se obtiene RCE en un servidor IIS via file upload sin restricciones. Se sube una webshell y se obtiene una reverse shell. Para escalar privilegios se necesita transferir `PrintSpoofer.exe`:

* **PowerShell bloqueado** por Application Control Policy
* **GitHub, Dropbox, Google Drive bloqueados** por filtrado de contenido web
* **Puerto 21 (FTP) bloqueado** por el firewall de salida
* **Puerto 445 (SMB) permitido** → se usa `impacket-smbserver` → éxito

Un pentester que solo conoce un método de transferencia se habría quedado bloqueado en el tercer intento. Conocer el abanico completo de opciones permite adaptarse a cualquier restricción.

### Qué puede bloquear una transferencia

| Control                                  | Qué bloquea                                                            |
| ---------------------------------------- | ---------------------------------------------------------------------- |
| **Application Whitelisting**             | Binarios no autorizados (`certutil.exe`, `bitsadmin.exe`...)           |
| **AV / EDR**                             | Herramientas de pentest conocidas, payloads con firmas                 |
| **Firewall de red**                      | Puertos específicos (21, 445, 3389...)                                 |
| **Proxy / Web Filtering**                | Dominios externos (GitHub, pastebin...), tipos de archivo (.exe, .ps1) |
| **IDS / IPS**                            | Patrones de tráfico anómalos, transferencias grandes                   |
| **PowerShell Constrained Language Mode** | Muchos cmdlets de descarga                                             |

### Estructura del módulo

| Página                            | Contenido                                            |
| --------------------------------- | ---------------------------------------------------- |
| **Windows File Transfer Methods** | PowerShell, SMB, FTP, WebDAV — descarga y subida     |
| **Linux File Transfer Methods**   | wget, curl, SCP, /dev/tcp, servidores web rápidos    |
| **Transferring Files with Code**  | Python, PHP, Ruby, Perl, JavaScript — one-liners     |
| **Miscellaneous Methods**         | Netcat, Ncat, RDP, Impacket, TFTP                    |
| **Protected File Transfers**      | Cifrado de archivos para transferencias seguras      |
| **Catching Files over HTTP/S**    | Nginx y Apache como receptores de uploads            |
| **Living off The Land**           | LOLBins — usar binarios del sistema para transferir  |
| **Detección y Evasión**           | Cómo se detectan las transferencias y cómo evadirlas |
