---
icon: shield-halved
---

# Analista de SOC - Responsabilidades

### ¿Qué es un analista de SOC?

El analista de SOC es la primera persona en investigar las amenazas a un sistema. Cuando el SIEM genera una alerta, es el analista quien determina si representa una amenaza real o es un falso positivo. Si la situación lo requiere, escala el incidente a sus supervisores para que puedan mitigar la amenaza.

```
Posición en el flujo de trabajo:

Sistema genera evento
       ↓
SIEM correlaciona y genera alerta
       ↓
ANALISTA SOC ← aquí empieza el trabajo
       ↓
Investigación con herramientas SOC
       ↓
Conclusión: FP o amenaza real
       ↓
Si amenaza: escalado + respuesta
```

### Rutina de un analista SOC

```
Durante el turno, el analista normalmente:

1. Revisa el panel del SIEM al comenzar el turno
   → ¿Hay alertas críticas pendientes?
   → ¿Qué quedó abierto del turno anterior?

2. Gestiona las alertas del canal principal
   → Toma posesión de la alerta (para que sus compañeros sepan que ya la atiende)
   → Analiza los detalles con las herramientas del SOC
   → Determina si es FP o amenaza real
   → Documenta la conclusión y las acciones tomadas

3. Coordina con el equipo
   → Comunica incidentes activos a los compañeros
   → Escala cuando es necesario
   → Recibe el briefing de incidentes del turno anterior

4. Contribuye a la mejora continua
   → Reporta al equipo de ingeniería SIEM cuando detecta FPs recurrentes
   → Sugiere ajustes en las reglas para reducir ruido
```

### Responsabilidades detalladas

#### Monitorización continua

```
→ Revisar el panel del SIEM durante todo el turno
→ Priorizar alertas según su severidad (Critical > High > Medium > Low)
→ No dejar alertas sin revisar al terminar el turno
→ Verificar que no hay alertas "huérfanas" (nadie las está investigando)
```

#### Triaje de alertas

```
El triaje es la clasificación inicial rápida de la alerta:
→ ¿Es un FP conocido? → cerrar con justificación
→ ¿Requiere investigación? → tomar posesión e investigar
→ ¿Es claramente crítico? → escalar inmediatamente mientras se investiga

Información mínima a recopilar en el triaje:
□ ¿Qué regla se disparó?
□ IP origen y destino
□ Usuario involucrado
□ Sistema afectado
□ Timestamp del evento
□ ¿Ha ocurrido antes? (buscar en historial)
```

#### Investigación con herramientas SOC

```
Para llegar a una conclusión, el analista usa múltiples herramientas:

EDR (Endpoint Detection & Response):
→ Ver el árbol de procesos del sistema afectado
→ ¿Qué proceso generó el evento? ¿Qué lo lanzó?
→ ¿Ha hecho conexiones de red inusuales?
→ ¿Ha creado o modificado archivos?

Log Management:
→ Buscar la IP o usuario en los logs históricos
→ ¿Ha aparecido antes? ¿Con qué frecuencia?
→ ¿Ha accedido a otros sistemas?

Threat Intelligence:
→ Consultar la IP/dominio/hash en VirusTotal, AbuseIPDB...
→ ¿Es un IOC conocido? ¿Con qué malware se asocia?
→ ¿Es una IP de un proveedor legítimo o de un C2?

SOAR:
→ Puede enriquecer automáticamente la alerta con contexto
→ Ejecutar acciones automáticas (bloquear IP, abrir ticket)
```

#### Documentación

```
Cada alerta investigada debe documentarse:
→ Qué se encontró
→ Qué herramientas se consultaron y qué mostraron
→ Por qué se determinó que es FP o amenaza real
→ Qué acciones se tomaron
→ Recomendaciones para evitar recurrencia

Una documentación pobre hace al SOC menos efectivo:
→ El siguiente turno no tiene contexto
→ Es imposible detectar patrones en incidentes relacionados
→ La mejora continua de las reglas SIEM se vuelve imposible
```

#### Escalado

```
¿Cuándo escalar?
→ La alerta supera la capacidad técnica del analista actual
→ El incidente parece ser una amenaza real grave
→ Hay múltiples sistemas afectados
→ Hay indicios de movimiento lateral o exfiltración
→ El analista tiene dudas y prefiere una segunda opinión

¿A quién escalar?
→ L1 → L2: análisis más profundo necesario
→ L2 → L3: incidente complejo, malware avanzado, APT
→ L3 → Incident Response: incidente activo confirmado
→ Directamente a IR: ransomware, brecha de datos confirmada

Cómo escalar correctamente:
→ Pasar TODO el contexto recopilado hasta ese momento
→ No escalar sin documentar lo que ya se investigó
→ Indicar claramente por qué se escala y cuál es el nivel de urgencia
```

### Habilidades necesarias del analista SOC

#### Sistemas operativos

```
Windows:
→ Conocer los procesos legítimos del sistema (lsass.exe, svchost.exe, etc.)
→ Saber qué Event IDs son relevantes para la seguridad
→ Entender Active Directory básico (usuarios, grupos, GPOs)
→ Conocer los mecanismos de persistencia del malware
→ PowerShell básico para consultas forenses

Linux:
→ Familiaridad con la estructura de directorios y logs
→ Comandos de análisis: grep, awk, netstat, ps, lsof
→ Conocer los archivos de configuración importantes
→ Saber cómo los atacantes obtienen persistencia en Linux
```

#### Redes

```
→ Comprender el modelo OSI y TCP/IP
→ Conocer los puertos y protocolos comunes
→ Saber distinguir tráfico normal de tráfico sospechoso
→ Entender qué es el beaconing de C2
→ Leer e interpretar logs de firewall e IDS
→ Conceptos de DNS, HTTP/HTTPS, SMB, Kerberos
```

#### Análisis de malware básico

```
→ Usar sandboxes para analizar el comportamiento de un archivo
→ Leer el árbol de procesos y entender lo que muestra
→ Identificar IOCs en el análisis: IPs C2, dominios, hashes
→ Entender los tipos de malware: RAT, keylogger, ransomware, dropper...
→ Saber qué información extraer para la investigación:
  → ¿A qué C2 se conecta?
  → ¿Qué persistencia instala?
  → ¿Qué datos roba o cifra?
```

#### Threat Intelligence

```
→ Saber usar VirusTotal, AbuseIPDB, URLScan.io, Shodan
→ Entender qué son los IOCs y cómo usarlos
→ Conocer MITRE ATT&CK y usarlo para contextualizar los hallazgos
→ Distinguir entre información relevante y ruido de internet
```

### Ventajas de ser analista SOC

```
Variedad de incidentes:
→ Cada alerta es distinta: phishing, bruteforce, malware, APT...
→ El trabajo es investigativo, no mecánico si se hace bien
→ Se aprende constantemente de nuevos tipos de ataques

Puerta de entrada al sector:
→ Es uno de los roles más accesibles para quien empieza
→ Permite construir experiencia real con amenazas reales
→ Desde L1 se puede escalar a L2, L3, IR, Threat Hunter, Red Team...

Impacto real:
→ Un buen analista puede detectar un ataque antes de que cause daño
→ Protege los datos de miles de personas
→ El trabajo tiene consecuencias tangibles para la organización
```

> Un buen analista SOC no depende de los productos de seguridad para pensar — los usa para obtener datos, pero **la interpretación la hace él**. Saber qué buscar y por qué es más valioso que saber manejar el SIEM.

> La habilidad más diferenciadora de un analista es **reconocer el comportamiento normal del sistema** para poder identificar lo que no lo es. Antes de buscar anomalías, hay que saber qué es normal.

> El analista SOC es la primera línea de defensa pero no la última. Escalar correctamente y a tiempo es tan importante como investigar bien. Un incidente detectado pero no escalado a tiempo puede ser tan dañino como uno no detectado.
