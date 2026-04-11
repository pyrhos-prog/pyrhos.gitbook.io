# FTP - File Transfer Protocol

FTP (File Transfer Protocol) es uno de los protocolos más antiguos de internet, diseñado para transferir archivos entre un cliente y un servidor. Opera en los puertos **21** (control) y **20** (datos en modo activo). A pesar de su antigüedad, sigue siendo muy común en entornos empresariales, servidores de backup y sistemas legacy.

El problema fundamental de FTP es que transmite **todo en texto claro**, incluyendo credenciales. No fue diseñado con seguridad en mente.

### Modos de conexión

| Modo       | Comportamiento                                                                                                                      |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Activo** | El cliente abre un puerto aleatorio y el servidor conecta de vuelta al cliente desde el puerto 20. Los firewalls suelen bloquearlo. |
| **Pasivo** | El servidor abre un puerto aleatorio alto y el cliente conecta a él. Más compatible con firewalls.                                  |

### Enumeración

#### Nmap

```bash
# Escaneo básico del puerto FTP
nmap -sV -p 21 target

# Scripts de nmap específicos para FTP
nmap -sC -sV -p 21 target

# Scripts específicos
nmap --script ftp-anon,ftp-banner,ftp-syst,ftp-brute -p 21 target

# Comprobar si acepta login anónimo
nmap --script ftp-anon -p 21 target
```

#### Login anónimo

El **anonymous login** es una de las misconfiguraciones más frecuentes. Muchos administradores lo habilitan para facilitar el acceso a archivos públicos y olvidan restringirlo.

```bash
# Conectar con usuario anonymous
ftp target
# Username: anonymous
# Password: (vacío o cualquier email)

# O directamente:
ftp anonymous@target
```

#### Comandos FTP útiles

Una vez conectado al servidor FTP, los comandos más útiles para enumeración:

```
ls -la          # listar archivos (incluyendo ocultos)
pwd             # directorio actual
cd nombre       # cambiar directorio
get archivo     # descargar un archivo
mget *          # descargar todos los archivos
put archivo     # subir un archivo (si hay permisos)
binary          # cambiar a modo binario (para archivos no texto)
passive         # alternar modo pasivo
status          # ver estado de la conexión
```

#### Descarga recursiva

```bash
# Con wget — descargar todo el contenido del FTP
wget -m --no-passive ftp://anonymous:anonymous@target

# Con lftp
lftp -c "open -u anonymous, ftp://target; mirror --parallel=10 / ./ftp-content/"
```

### Riesgos y misconfiguraciones

| Riesgo                           | Descripción                                                                                                              |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Anonymous login**              | Permite acceso sin credenciales. Si el directorio es escribible, el atacante puede subir archivos (webshells, payloads). |
| **Credenciales en claro**        | FTP no cifra nada. En un entorno con MITM, usuario y contraseña son capturables con Wireshark.                           |
| **Permisos de escritura**        | Si el usuario anónimo o un usuario con credenciales débiles puede escribir, se pueden subir archivos maliciosos.         |
| **FTP Bounce Attack**            | Usar el servidor FTP para escanear otros hosts internos o establecer conexiones a terceros.                              |
| **Versiones antiguas**           | Versiones de vsftpd, ProFTPD o Pure-FTPd con vulnerabilidades conocidas (CVEs).                                          |
| **Archivos sensibles expuestos** | Backups, archivos de configuración, credenciales de base de datos en el directorio FTP.                                  |

#### Captura de credenciales con Wireshark

```
Filtro Wireshark para ver credenciales FTP:
ftp.request.command == "USER" || ftp.request.command == "PASS"
```

### Ataques

#### Brute Force

```bash
# Hydra
hydra -l usuario -P /usr/share/wordlists/rockyou.txt ftp://target
hydra -L users.txt -P passwords.txt ftp://target -t 10

# Medusa
medusa -h target -u admin -P /usr/share/wordlists/rockyou.txt -M ftp
```



#### FTP Bounce Attack

Es una técnica en la que se abusa de un servidor FTP mal configurado para que actúe como intermediario y envíe conexiones o datos a otros sistemas.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

**Cómo funciona**

1. El atacante se conecta a un servidor FTP vulnerable.
2. Usa comandos como `PORT` para indicar una **IP y puerto de un tercero**.
3. El servidor FTP intenta conectarse a ese tercero.

**Para qué se usa**

* Ocultar la IP real del atacante.
* Saltarse filtros/firewalls.
* Reconocimiento de red (escaneo).

**Ejemplo**

```bash
# Bounce Attack con Nmap para escanear el puerto 80 de una IP victima
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

* El parametro -b se utiliza para hacer el bounce attack

### FTPS y SFTP

| Protocolo | Descripción                                                                                            |
| --------- | ------------------------------------------------------------------------------------------------------ |
| **FTPS**  | FTP sobre TLS/SSL. Cifra la comunicación pero sigue usando los puertos 21/20.                          |
| **SFTP**  | SSH File Transfer Protocol. No tiene relación con FTP — usa SSH (puerto 22). Es la alternativa segura. |

> Encontrar un FTP con **anonymous login y permisos de escritura** es un hallazgo crítico. Se puede subir una webshell si hay un servidor web en el mismo servidor apuntando a ese directorio, o usar el acceso para pivotar.

> Al enumerar FTP, revisar siempre los **directorios ocultos** con `ls -la`. Los administradores a veces colocan archivos sensibles en subdirectorios creyendo que no son accesibles con login anónimo.
