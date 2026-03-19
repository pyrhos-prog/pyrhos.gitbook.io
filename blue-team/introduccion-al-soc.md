---
icon: shield-halved
---

# Introducción al SOC

### ¿Qué es un SOC?

Un **Security Operations Center (SOC)** es la instalación donde el equipo de seguridad de una organización monitoriza y analiza continuamente su postura de seguridad.&#x20;

El objetivo principal del equipo SOC es **detectar, analizar y responder** a incidentes de ciberseguridad utilizando personas, procesos y tecnología de forma coordinada.

```
El SOC actúa como el sistema inmune de la organización:
→ Monitoriza en tiempo real (24/7/365)
→ Detecta amenazas antes de que causen daño
→ Responde y contiene los incidentes cuando ocurren
→ Aprende de cada ataque para mejorar las defensas
```

### Los tres pilares del SOC

Un SOC efectivo se apoya en la coordinación de tres elementos. Ninguno funciona sin los otros dos.

#### Personas

```
→ Analistas con formación técnica y experiencia en incidentes
→ Capacidad de adaptarse a nuevos tipos de ataques
→ Mentalidad investigadora: no aceptar las cosas en su valor facial
→ Conocimiento actualizado de las técnicas de los atacantes
→ Trabajo en equipo y comunicación clara bajo presión
```

#### Procesos

```
→ Procedimientos estandarizados (playbooks) para cada tipo de incidente
→ Flujos claros de escalado entre niveles (L1 → L2 → L3 → IR)
→ Alineación con frameworks de seguridad: NIST, PCI-DSS, ISO 27001
→ Documentación rigurosa de cada acción tomada
→ Revisión continua: cada incidente es una oportunidad de mejorar
```

#### Tecnología

```
→ SIEM       → correlación de eventos y generación de alertas
→ EDR        → visibilidad y respuesta en endpoints
→ Log Mgmt   → almacenamiento y búsqueda de logs históricos
→ SOAR       → automatización de la respuesta a incidentes
→ Threat Intel → contexto sobre amenazas activas
→ IDS/IPS    → detección y prevención de intrusiones en red
```

### ¿Por qué existe el SOC?

Las organizaciones necesitan un SOC porque:

```
1. Los ataques son constantes y automatizados
   → Los atacantes usan bots que prueban vulnerabilidades 24/7
   → Sin monitorización continua, una brecha puede pasar semanas sin detectarse

2. El tiempo de detección importa
   → Cada hora que un atacante está en la red sin ser detectado
     puede significar más datos robados, más sistemas comprometidos
   → MTTD (Mean Time To Detect) y MTTR (Mean Time To Respond)
     son los indicadores más críticos de la efectividad del SOC

3. La complejidad de los entornos modernos
   → Endpoints, cloud, aplicaciones web, APIs, IoT, trabajo remoto...
   → Ningún humano puede monitorizar todo manualmente
   → El SIEM correlaciona millones de eventos y genera alertas priorizadas

4. Requisitos normativos
   → RGPD, PCI-DSS, ISO 27001, ENS (España)...
   → Muchas normativas exigen capacidades de detección y respuesta
```

### Métricas clave del SOC

```
MTTD — Mean Time To Detect:
→ Tiempo desde que ocurre el incidente hasta que se detecta
→ Objetivo: minimizar. Los atacantes avanzan mientras no son detectados.
→ Media global actual: ~21 días (muchos atacantes APT)
→ SOC maduro: horas o minutos

MTTR — Mean Time To Respond:
→ Tiempo desde la detección hasta la contención completa
→ Cuanto menor, menor el daño causado

Tasa de Falsos Positivos:
→ % de alertas que no son amenazas reales
→ Alta tasa → analistas saturados → se pasan cosas por alto (alert fatigue)

Cobertura MITRE ATT&CK:
→ % de técnicas de ataque conocidas para las que hay detección activa
→ Los gaps = puntos ciegos del SOC
```

### El ciclo de vida de una alerta en el SOC

```
EVENTO ocurre en el sistema
        ↓
Log generado → enviado al SIEM
        ↓
SIEM aplica reglas de correlación
        ↓
ALERTA generada (si coincide con una regla)
        ↓
Analista L1 → TRIAJE
  ¿Falso positivo o amenaza real?
        ↓
   Si amenaza: INVESTIGACIÓN (L1/L2)
        ↓
   Si complejo: ESCALADO a L2/L3
        ↓
RESPUESTA: contención, erradicación, recuperación
        ↓
DOCUMENTACIÓN y mejora de las reglas
```

### Herramientas principales del analista SOC

```
SIEM (Security Information and Event Management):
→ Panel principal del analista
→ Muestra todas las alertas en tiempo real
→ Ejemplos: Microsoft Sentinel, Splunk, IBM QRadar, Wazuh

EDR (Endpoint Detection & Response):
→ Visibilidad en profundidad de lo que ocurre en cada equipo
→ Árbol de procesos, conexiones de red, operaciones de archivos
→ Ejemplos: CrowdStrike, SentinelOne, Microsoft Defender for Endpoint

Log Management:
→ Almacenamiento y búsqueda de logs históricos
→ Para investigaciones forenses y búsqueda de patrones
→ Ejemplos: Elastic Stack, Splunk

SOAR:
→ Automatiza tareas repetitivas del SOC
→ Enriquece alertas con contexto de Threat Intelligence
→ Abre tickets, bloquea IPs, notifica al equipo automáticamente

Threat Intelligence:
→ Información sobre IOCs (IPs, dominios, hashes maliciosos)
→ Contexto sobre grupos de atacantes y sus técnicas
→ Ejemplos: VirusTotal, AbuseIPDB, MITRE ATT&CK
```

> El SOC no es solo tecnología. Un SIEM con reglas perfectas pero operado por un equipo sin experiencia fallará igualmente. La combinación personas + procesos + tecnología es lo que hace funcionar un SOC de verdad.

> Alert fatigue (fatiga de alertas) es el mayor enemigo del SOC. Demasiados falsos positivos hacen que los analistas empiecen a ignorar alertas — y ahí es cuando los ataques reales pasan desapercibidos. Mantener la tasa de FP baja es tan importante como detectar amenazas.

> Un SOC que detecta incidentes semanas después de que ocurrieron tiene el mismo problema que no tener SOC. La velocidad de detección (MTTD) es el indicador más importante de su efectividad.

> El impacto de los ataques de red (ARP spoofing, VLAN hopping) puede ser alto y visible. En entornos de producción, coordinar siempre con el cliente antes de lanzarlos.
