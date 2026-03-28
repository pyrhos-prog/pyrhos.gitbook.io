---
icon: shield-halved
---

# Relación SIEM y Analista

**SIEM** (Security Information and Event Management) es la solución central del SOC. Combina la recopilación y almacenamiento de logs a largo plazo (SIM) con la monitorización en tiempo real y generación de alertas (SEM). Su objetivo final es **detectar amenazas** a través del registro y correlación de eventos.

### Cómo funciona el SIEM

El SIEM recibe logs de toda la infraestructura (endpoints, servidores, red, cloud, identidad, herramientas de seguridad), los normaliza a un formato unificado, aplica reglas de correlación y genera alertas cuando detecta patrones sospechosos.

```
Fuentes de logs → SIEM → Normalización → Correlación → Alerta → Analista
```

### El analista frente al SIEM

Aunque el SIEM tiene muchas características avanzadas, el trabajo diario del analista L1 se centra en:

1. **Monitorizar el panel de alertas** — revisar el canal principal continuamente
2. **Tomar posesión de una alerta** — hacer clic en "Take Ownership" para que los compañeros sepan que ya está siendo trabajada y evitar duplicar esfuerzo
3. **Revisar los detalles** — el SIEM muestra la información básica (IP, hostname, usuario, hash...) con la que el analista va a las otras herramientas
4. **Concluir y documentar** — registrar la conclusión en el SIEM: FP con justificación o escalar como amenaza real

> El SIEM es el **punto de partida** de la investigación, no el único lugar donde se investiga. Los datos del SIEM llevan al analista a consultar el EDR, los logs, la Threat Intelligence... El SIEM muestra que algo pasó — el analista descubre qué pasó realmente.

### Reglas de correlación — cómo se generan las alertas

Las reglas definen cuándo se genera una alerta. Son el corazón del SIEM.

| Tipo de regla                 | Descripción                                   | Ejemplo                                  |
| ----------------------------- | --------------------------------------------- | ---------------------------------------- |
| **Umbral**                    | Más de N eventos en Y segundos                | 20 logins fallidos en 10s → bruteforce   |
| **Secuencia**                 | Evento A seguido de Evento B                  | Escaneo de puertos + intento de conexión |
| **Correlación entre fuentes** | La misma IP aparece en firewall Y en IDS      | Mayor precisión al cruzar datos          |
| **Anomalía (UEBA)**           | El usuario se comporta distinto a su baseline | Empleado conectándose a las 3am          |
| **Lista negra**               | Comunicación con IOC conocido                 | IP maliciosa de un feed de Threat Intel  |

### Falsos positivos — el mayor reto del SIEM

Un **falso positivo (FP)** es una alerta generada por el SIEM que no corresponde a ninguna amenaza real. Ocurren porque las reglas son demasiado genéricas o no contemplan excepciones para actividades legítimas.

**Ejemplo real:** una regla que alerta cuando una URL contiene la palabra "union" (para detectar SQL Injection). Un usuario busca en Google `https://www.google.com/search?q=sql+union+usage` y se genera una alerta — no hay ninguna inyección real.

El analista identifica el FP, lo documenta y **reporta al equipo de ingeniería SIEM** para que ajusten la regla. Nunca se debe ajustar la regla por cuenta propia sin comunicarlo.

> Cuando el analista identifica un falso positivo recurrente, **comunicarlo siempre al equipo** en lugar de simplemente cerrarlo. Un FP no reportado seguirá generándose indefinidamente, consumiendo tiempo del equipo y contribuyendo a la fatiga de alertas.

### Soluciones SIEM comunes

| SIEM                   | Lenguaje de consulta | Características                                                   |
| ---------------------- | -------------------- | ----------------------------------------------------------------- |
| **Microsoft Sentinel** | KQL                  | Cloud-native, excelente integración con Azure y M365              |
| **Splunk**             | SPL                  | Muy potente para búsquedas complejas, gran ecosistema             |
| **IBM QRadar**         | AQL                  | Clásico en enterprise y sector financiero                         |
| **Elastic SIEM**       | KQL / EQL            | Open source en versión básica, muy flexible                       |
| **Wazuh**              | —                    | 100% gratuito, ideal para aprender, integración con MITRE ATT\&CK |

### Quién configura el SIEM

El analista L1 normalmente **no toca la configuración del SIEM**. Esa responsabilidad recae en:

* **Ingeniero de Seguridad SOC** → crea y ajusta las reglas, integra fuentes de logs, configura dashboards
* **Analista L3 / Threat Hunter** → crea nuevas reglas de detección basadas en TTPs, ajusta reglas para reducir FPs

Lo que el analista L1 sí puede y debe hacer: reportar FPs recurrentes, sugerir ajustes basados en patrones observados, y dar feedback al equipo de ingeniería.

> La calidad de un SIEM depende directamente de la calidad de sus regla&#x73;**.** Un SIEM con reglas mal configuradas genera alertas constantemente (fatiga de alertas) o no detecta amenazas reales. El feedback del analista al equipo de ingeniería es fundamental para mejorar las reglas.
