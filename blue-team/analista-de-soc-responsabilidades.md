---
icon: shield-halved
---

# Analista de SOC - Responsabilidades

El analista de SOC es la **primera persona en investigar las amenazas** a un sistema. Cuando el SIEM genera una alerta, es el analista quien determina si representa una amenaza real o un falso positivo. Si la situación lo requiere, escala el incidente a sus supervisores.

### Rutina diaria de un analista de SOC

1. Al comenzar el turno, el analista revisa el panel del SIEM para ver si hay alertas críticas pendientes o casos abiertos del turno anterior.&#x20;
2. Durante el turno gestiona las alertas del canal principal: toma posesión de cada alerta para que sus compañeros sepan que ya la atiende, la analiza con las herramientas del SOC, determina si es FP o amenaza real, y documenta la conclusión con las acciones tomadas.&#x20;
3. Al final del turno realiza el briefing de traspaso al equipo siguiente.

### Responsabilidades detalladas

#### Monitorización continua

Revisar el panel del SIEM durante todo el turno, priorizar alertas según severidad (Critical > High > Medium > Low) y asegurarse de que no hay alertas sin revisar ni "huérfanas" (sin nadie investigándolas).

#### Triaje de alertas

El triaje es la clasificación inicial rápida:

* ¿Es un FP conocido? → cerrar con justificación
* ¿Requiere investigación? → tomar posesión e investigar
* ¿Es claramente crítico? → escalar inmediatamente **mientras** se investiga

La información mínima a recopilar en el triaje: qué regla se disparó, IP origen y destino, usuario involucrado, sistema afectado, timestamp del evento, y si ha ocurrido antes.

#### Investigación con herramientas SOC

Para llegar a una conclusión el analista usa múltiples herramientas en paralelo:

| Herramienta             | Qué se busca                                                       |
| ----------------------- | ------------------------------------------------------------------ |
| **EDR**                 | Árbol de procesos, conexiones de red, archivos creados/modificados |
| **Log Management**      | Historial de la IP o usuario en los últimos días                   |
| **Threat Intelligence** | Reputación de la IP/dominio/hash en VirusTotal, AbuseIPDB...       |
| **SOAR**                | Enriquecimiento automático y acciones básicas ya ejecutadas        |

#### Escalado correcto

¿Cuándo escalar? Cuando la alerta supera la capacidad técnica del analista, cuando hay múltiples sistemas afectados, cuando hay indicios de movimiento lateral o exfiltración, o cuando hay dudas y se prefiere una segunda opinión.

Cómo escalar correctamente: pasar **todo el contexto** recopilado hasta ese momento, indicar qué se investigó y qué se encontró, y explicar claramente por qué se escala y cuál es el nivel de urgencia estimado.

### Habilidades necesarias

#### Sistemas operativos

Conocer qué es normal en Windows y Linux es fundamental para detectar lo que no lo es. En Windows: procesos legítimos del sistema, Event IDs relevantes para seguridad, mecanismos de persistencia del malware. En Linux: estructura de directorios y logs, comandos de análisis (`grep`, `awk`, `netstat`, `ps`, `lsof`).

#### Redes

Comprender el modelo OSI y TCP/IP, conocer los puertos y protocolos comunes, saber distinguir tráfico normal de tráfico sospechoso, entender qué es el beaconing de C2, y saber leer e interpretar logs de firewall e IDS.

#### Análisis de malware básico

Usar sandboxes para analizar el comportamiento de un archivo sospechoso, leer el árbol de procesos, identificar IOCs en el análisis (IPs C2, dominios, hashes), y entender los tipos de malware más comunes: RAT, keylogger, ransomware, dropper.

#### Threat Intelligence

Saber usar VirusTotal, AbuseIPDB, URLScan.io y Shodan. Entender qué son los IOCs y cómo usarlos. Conocer MITRE ATT\&CK para contextualizar los hallazgos.

> Un buen analista SOC no depende de los productos de seguridad para pensar — los usa para obtener datos, pero **la interpretación la hace él**. Saber qué buscar y por qué es más valioso que saber manejar el SIEM.

> El analista SOC es la primera línea de defensa pero no la última. Escalar correctamente y a tiempo es tan importante como investigar bien. Un incidente detectado pero no escalado a tiempo puede ser tan dañino como uno no detectado.
