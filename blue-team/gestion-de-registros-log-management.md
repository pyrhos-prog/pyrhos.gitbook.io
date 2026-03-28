---
icon: shield-halved
---

# Gestión de Registros (Log Management)

Un **log** es un registro cronológico de eventos que ocurren en un sistema, aplicación o dispositivo de red. Cada acción relevante genera una entrada con información sobre qué ocurrió, cuándo, dónde y quién lo hizo.

Sin logs es imposible detectar qué ocurrió en el sistema, hacer análisis forense post-incidente, determinar el alcance de una brecha o cumplir con normativas como el RGPD o PCI-DSS. Con logs bien centralizados, el SIEM puede correlacionar eventos, el analista puede investigar cualquier incidente histórico y se puede reconstruir la cadena de ataque completa.

### Windows Event IDs esenciales

#### Autenticación y acceso

| Event ID | Descripción                           | Por qué importa                                                   |
| -------- | ------------------------------------- | ----------------------------------------------------------------- |
| `4624`   | Logon exitoso                         | Ver el tipo de logon (2=interactivo, 3=red, 10=RDP, 7=desbloqueo) |
| `4625`   | Logon fallido                         | Bruteforce, credenciales incorrectas                              |
| `4634`   | Logoff                                | —                                                                 |
| `4648`   | Logon con credenciales explícitas     | Uso de `runas`, sospechoso si es frecuente                        |
| `4672`   | Privilegio especial asignado al logon | Indica logon de administrador                                     |

#### Gestión de cuentas

| Event ID | Descripción                                 |
| -------- | ------------------------------------------- |
| `4720`   | Cuenta de usuario creada                    |
| `4722`   | Cuenta habilitada                           |
| `4723`   | Cambio de contraseña (el propio usuario)    |
| `4724`   | Reset de contraseña (por otro usuario)      |
| `4728`   | Usuario añadido a grupo global de seguridad |
| `4732`   | Usuario añadido a grupo local               |

#### Procesos y ejecución

| Event ID | Descripción                                       | Por qué importa                                                                 |
| -------- | ------------------------------------------------- | ------------------------------------------------------------------------------- |
| `4688`   | Nuevo proceso creado                              | Con CommandLine logging: muestra el comando exacto. Crítico para detectar LOtL. |
| `4689`   | Proceso terminado                                 | —                                                                               |
| `4103`   | Ejecución de módulo PowerShell                    | —                                                                               |
| `4104`   | Bloque de script PowerShell (ScriptBlock Logging) | Muestra el código PowerShell completo, incluyendo código ofuscado               |

#### Servicios, tareas y Active Directory

| Event ID | Descripción                        | Por qué importa                                  |
| -------- | ---------------------------------- | ------------------------------------------------ |
| `7045`   | Nuevo servicio instalado           | PsExec, malware persistente                      |
| `4698`   | Tarea programada creada            | Persistencia de malware                          |
| `4768`   | TGT de Kerberos solicitado         | —                                                |
| `4769`   | TGS de Kerberos solicitado         | Múltiples en poco tiempo → posible Kerberoasting |
| `4771`   | Pre-autenticación Kerberos fallida | AS-REP Roasting o bruteforce                     |
| `4776`   | Validación de credenciales NTLM    | Pass-the-Hash                                    |

> Activar el **logging de la línea de comandos** en el Event ID `4688` y **PowerShell ScriptBlock Logging** (`4104`) es una de las mejoras más impactantes que se puede hacer en la detección. Sin ellos, los ataques Living off the Land son casi invisibles.

### Logs de Linux esenciales

Los archivos de log más importantes para el analista SOC en Linux:

* `/var/log/auth.log` (Debian/Ubuntu) o `/var/log/secure` (RHEL/CentOS) — intentos de login, sudo, SSH
* `/var/log/syslog` — eventos generales del sistema
* `/var/log/apache2/access.log` o `/var/log/nginx/access.log` — cada petición HTTP al servidor web

Comandos útiles de análisis:

```bash
# Top IPs con más intentos fallidos de SSH
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Historial de logins exitosos / fallidos
last
lastb

# Ver usuarios conectados ahora
who
```

### Logs de red

Los logs de red complementan los de los sistemas con una perspectiva diferente: el tráfico que entra y sale.

* **Firewall logs** — tráfico permitido y bloqueado. El tráfico saliente inusual puede indicar exfiltración o C2.
* **IDS/IPS logs** (Snort, Suricata) — alertas de firmas de ataque conocidas y anomalías.
* **Proxy / Web gateway logs** — URLs accedidas por los usuarios. Peticiones a dominios maliciosos o inusuales.
* **DNS logs** — cada consulta DNS de la red. Fundamentales para detectar dominios de C2, DNS tunneling y dominios DGA.
* **VPN logs** — conexiones de usuarios remotos. Usuarios conectados fuera de horario o desde países inusuales.

> Los **logs de DNS** son una fuente de detección muy subestimada. Un malware que se conecta a su C2 siempre hace una consulta DNS primero. Los logs de DNS revelan comunicaciones maliciosas incluso cuando el tráfico HTTP/HTTPS está cifrado.

### Centralización de logs

| Sin centralización                                          | Con centralización                         |
| ----------------------------------------------------------- | ------------------------------------------ |
| Logs en cada sistema por separado                           | Todos los logs en un único lugar           |
| Investigar requiere ir máquina por máquina                  | Búsqueda global en toda la infraestructura |
| Si el atacante borra los logs locales → evidencias perdidas | Los logs ya están en el servidor central   |
| Imposible correlacionar eventos entre sistemas              | El SIEM correlaciona en tiempo real        |

### Elastic Stack (ELK)

Elastic Stack es el sistema de gestión de logs más extendido:

* **Elasticsearch** — motor de búsqueda y almacenamiento
* **Logstash** — pipeline de ingestión y transformación
* **Kibana** — interfaz web para visualización y búsqueda
* **Beats** — agentes ligeros de recolección (`Filebeat`, `Winlogbeat`, `Packetbeat`)

```bash
# Búsqueda en Kibana (KQL)
event.code: "4625" and winlog.event_data.IpAddress: "45.33.32.156"

event.code: "4688" and process.command_line: "*powershell*-enc*"
```

### Retención de logs — normativa

| Normativa          | Retención mínima                                           |
| ------------------ | ---------------------------------------------------------- |
| **PCI-DSS**        | 12 meses (3 meses en línea / acceso inmediato)             |
| **ENS (España)**   | 6 meses (básico) — 5 años (alto)                           |
| **RGPD**           | No especifica, pero deben estar disponibles para auditoría |
| **Buena práctica** | 90 días en línea + 1 año en archivo                        |

> Los logs deben enviarse a un **sistema de almacenamiento centralizado en tiempo real**. Si el atacante compromete un sistema y borra sus logs locales, los ya copiados al servidor central siguen disponibles.
