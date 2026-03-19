---
icon: shield-halved
---

# Gestión de Registros (Log Management)

### ¿Qué es un log?

Un **log** (registro) es un registro cronológico de eventos que ocurren en un sistema, aplicación o dispositivo de red. Cada acción relevante genera una entrada en el log con información sobre qué ocurrió, cuándo, dónde y quién lo hizo.

```
Ejemplo de log de Windows (Event ID 4625 — login fallido):
An account failed to log on.

Subject:    Security ID: NULL SID
            Account Name: -

Failure Information:
            Failure Reason: Unknown user name or bad password.
            Status: 0xC000006D
            Sub Status: 0xC0000064

Network Information:
            Workstation Name: DESKTOP-USER01
            Source Network Address: 192.168.1.50
            Source Port: 49812

Account For Which Logon Failed:
            Account Name: administrator
            Account Domain: EMPRESA
```

### ¿Por qué son críticos los logs para el SOC?

```
Sin logs:
→ No se puede detectar qué ocurrió en el sistema
→ Es imposible hacer análisis forense post-incidente
→ No hay evidencia para determinar el alcance de una brecha
→ Incumplimiento normativo (RGPD, PCI-DSS, ENS...)

Con logs:
→ El SIEM puede correlacionar eventos y generar alertas
→ El analista puede investigar qué pasó, cuándo y cómo
→ Se puede reconstruir la cadena de ataque (kill chain)
→ Se pueden detectar ataques que llevan tiempo en curso
→ Se cumple con los requisitos de auditoría y normativa
```

### Tipos de logs en el SOC

#### Logs de sistemas operativos

**Windows Event Logs**

```
Los logs más importantes para el analista SOC en Windows:

Autenticación y accesos:
4624 → Logon exitoso (importante: ver tipo de logon)
       Logon Type 2  = interactivo (teclado físico)
       Logon Type 3  = red (acceso a recurso compartido)
       Logon Type 10 = remoto interactivo (RDP)
       Logon Type 7  = desbloqueo de sesión
4625 → Logon fallido → bruteforce, credenciales incorrectas
4634 → Logoff
4648 → Logon con credenciales explícitas (runas)
4672 → Privilegio especial asignado (logon de admin)

Gestión de cuentas:
4720 → Cuenta de usuario creada
4722 → Cuenta habilitada
4723 → Cambio de contraseña (el propio usuario)
4724 → Reset de contraseña (por otro usuario)
4728 → Usuario añadido a grupo global de seguridad
4732 → Usuario añadido a grupo local
4756 → Usuario añadido a grupo universal

Procesos y ejecución:
4688 → Nuevo proceso creado
       → Con CommandLine logging activo: muestra el comando exacto
       → Crítico para detectar LOtL (Living off the Land)
4689 → Proceso terminado

Servicios y tareas:
7045 → Nuevo servicio instalado → PsExec, malware persistente
4698 → Tarea programada creada → persistencia de malware
4702 → Tarea programada modificada
4699 → Tarea programada eliminada

PowerShell:
4103 → Ejecución de módulo PowerShell
4104 → Bloque de script PowerShell (ScriptBlock Logging)
       → Muestra el código PowerShell completo ejecutado
       → Fundamental para detectar ofuscación

Active Directory y Kerberos:
4768 → TGT de Kerberos solicitado (AS-REQ)
4769 → Ticket de servicio Kerberos solicitado (TGS-REQ)
       → Múltiples en poco tiempo desde el mismo origen → Kerberoasting
4771 → Pre-autenticación Kerberos fallida → AS-REP Roasting o bruteforce
4776 → Validación de credenciales NTLM → Pass-the-Hash

Ubicación de los logs de Windows:
C:\Windows\System32\winevt\Logs\
→ Security.evtx   (eventos de seguridad)
→ System.evtx     (eventos del sistema)
→ Application.evtx (eventos de aplicaciones)
→ PowerShell/Operational.evtx (PowerShell)
→ Sysmon.evtx    (si Sysmon está instalado)
```

**Linux / Syslog**

```
Archivos de log principales:

/var/log/auth.log (Debian/Ubuntu) o /var/log/secure (RHEL/CentOS):
→ Intentos de login (SSH, sudo, PAM)
→ Autenticaciones exitosas y fallidas
→ Uso de sudo

Ejemplo de entrada en auth.log:
Jan 15 03:22:45 servidor sshd[1234]: Failed password for root
    from 45.33.32.156 port 52341 ssh2
Jan 15 03:22:47 servidor sshd[1234]: Failed password for root
    from 45.33.32.156 port 52342 ssh2
→ Dos fallos de SSH para root desde la misma IP → bruteforce

/var/log/syslog:
→ Eventos generales del sistema
→ Inicio/parada de servicios
→ Mensajes del kernel

/var/log/apache2/access.log o /var/log/nginx/access.log:
→ Cada petición HTTP al servidor web
→ Formato: IP - usuario [fecha] "método URL protocolo" código bytes
192.168.1.50 - - [15/Jan/2024:10:22:31] "GET /admin HTTP/1.1" 403 287
→ Muchos 403 desde la misma IP → reconocimiento o fuzzing

/var/log/apache2/error.log:
→ Errores del servidor web
→ Intentos de explotación suelen generar errores

Comandos de análisis:
grep "Failed password" /var/log/auth.log | tail -50
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
# → Top IPs con más intentos fallidos de SSH

last          # Historial de logins exitosos
lastb         # Historial de logins fallidos
who           # Usuarios conectados ahora
```

#### Logs de red

```
Firewall logs:
→ Tráfico permitido y bloqueado
→ IPs origen/destino, puertos, protocolo, acción
→ Importante: tráfico saliente inusual → posible exfiltración o C2

Ejemplo de log de firewall (iptables/pf style):
2024-01-15 03:22:45 DENY IN=eth0 SRC=45.33.32.156 DST=192.168.1.10
    PROTO=TCP DPT=22 FLAGS=SYN
→ Intento de conexión SSH desde IP externa bloqueado

IDS/IPS logs (Snort, Suricata):
→ Alertas de firmas de ataque conocidas
→ Anomalías en el tráfico de red
→ Ataques identificados por patrones

Proxy/Web gateway logs:
→ Todas las peticiones web de los usuarios
→ URLs accedidas, categorías, usuarios, bytes transferidos
→ Importante: peticiones a dominios maliciosos o inusuales

DNS logs:
→ Cada consulta DNS realizada en la red
→ Fundamental para detectar:
  → Dominios de C2
  → DNS tunneling (consultas con datos codificados)
  → DGA (Domain Generation Algorithm) del malware
  → Dominios recién registrados (indicador de phishing)

VPN logs:
→ Conexiones de usuarios remotos
→ IPs desde las que se conectan
→ Usuarios conectados fuera de horario o desde países inusuales
```

#### Logs de aplicaciones

```
Email logs (Exchange, Office 365):
→ Emails enviados y recibidos
→ Adjuntos y URLs en emails
→ Reglas de reenvío creadas (indicador de compromiso de cuenta)
→ Accesos al buzón desde IPs externas

Base de datos (MySQL, MSSQL, PostgreSQL):
→ Consultas ejecutadas
→ Usuarios que acceden a la DB
→ Errores de autenticación
→ Accesos fuera de horario

Aplicaciones web (NGINX, Apache, IIS):
→ Todas las peticiones HTTP/HTTPS
→ Errores 4xx/5xx
→ Parámetros de las peticiones (importante para detectar SQLi, XSS...)
```

### Gestión y almacenamiento de logs

#### Requisitos de retención

```
Normativa y retención mínima de logs:
→ RGPD:     no especifica tiempo, pero deben estar disponibles para auditoría
→ PCI-DSS:  mínimo 12 meses (3 meses en línea/acceso inmediato)
→ ENS (España): según categoría del sistema (básico: 6m, alto: 5 años)
→ ISO 27001: según análisis de riesgos de la organización
→ Buena práctica general: 90 días en línea, 1 año en archivo

Tipos de almacenamiento:
→ Hot storage:  logs recientes, acceso inmediato, costoso
→ Warm storage: logs de hace semanas, acceso en minutos
→ Cold storage: logs de hace meses/años, acceso en horas, económico
```

#### Elastic Stack (ELK) — el sistema de gestión de logs más común

```
Componentes:
Elasticsearch → motor de búsqueda y almacenamiento
Logstash      → pipeline de ingestión y transformación
Kibana        → interfaz web para visualización y búsqueda
Beats         → agentes ligeros de recolección (Filebeat, Winlogbeat, Packetbeat)

Flujo:
Sistema → Filebeat/Winlogbeat → Logstash (parsear/normalizar) → Elasticsearch → Kibana

Búsqueda en Kibana (KQL):
event.code: "4625" and winlog.event_data.IpAddress: "45.33.32.156"
→ Todos los logins fallidos desde esa IP

event.code: "4688" and process.command_line: "*powershell*-enc*"
→ Ejecuciones de PowerShell con comandos codificados (sospechoso)

@timestamp >= "2024-01-15T00:00:00" and source.ip: "192.168.1.50"
→ Todo el tráfico de esa IP en el día 15
```

### Log centralization — por qué es crítica

```
Sin centralización:
→ Los logs están en cada sistema por separado
→ Para investigar un incidente hay que ir máquina por máquina
→ Si el atacante borra los logs del sistema comprometido → evidencias perdidas
→ Imposible correlacionar eventos entre distintos sistemas

Con centralización (SIEM/Log Management):
→ Todos los logs en un único lugar
→ Búsqueda global en toda la infraestructura a la vez
→ Los logs del sistema comprometido ya están en el servidor central
   aunque el atacante los borre localmente
→ El SIEM puede correlacionar eventos de distintos sistemas

Arquitectura básica de centralización:
[Endpoint 1] → syslog/beats → [Log Collector] → [SIEM/Elasticsearch]
[Endpoint 2] →                                →
[Firewall]   →                                →
[AD/DC]      →                                →
```

### Búsqueda en logs durante una investigación

```
Cuando se investiga una alerta, el analista busca en los logs:

1. Contexto temporal: ¿qué más pasó alrededor del evento?
   → Buscar en ±30 minutos del timestamp de la alerta
   → ¿Hubo otros eventos relacionados antes o después?

2. Contexto del origen: ¿qué más ha hecho esta IP/usuario?
   → Buscar todos los eventos de esa IP en las últimas 24-48h
   → ¿Ha intentado acceder a otros sistemas?
   → ¿Es la primera vez que aparece esta IP?

3. Contexto del destino: ¿qué le ha pasado a este sistema?
   → Buscar todos los eventos del sistema afectado en las últimas horas
   → ¿Hay otros eventos sospechosos relacionados?

4. Correlación lateral: ¿hay otros sistemas afectados?
   → Buscar el IOC (IP, dominio, hash) en TODOS los sistemas
   → ¿Hay otros equipos comunicándose con ese C2?
   → ¿Hay más cuentas con actividad similar?
```

> Activar el logging de la línea de comandos en Windows (Event ID 4688 con CommandLine) y PowerShell Script Block Logging (Event ID 4104) es una de las mejoras más impactantes que se puede hacer en la detección. Sin ellos, los ataques LOtL son casi invisibles.

> Los logs de DNS son una fuente de detección subestimada. Un malware que se conecta a su C2 siempre hace una consulta DNS primero. Los logs de DNS revelan comunicaciones maliciosas incluso cuando el tráfico HTTP/HTTPS está cifrado.

> Los logs deben enviarse a un sistema de almacenamiento centralizado en tiempo real. Si el atacante compromete un sistema y borra sus logs locales, los logs ya copiados al servidor central siguen disponibles. Un log borrado localmente pero preservado en el SIEM es la diferencia entre reconstruir la cadena de ataque y no poder hacerlo.
