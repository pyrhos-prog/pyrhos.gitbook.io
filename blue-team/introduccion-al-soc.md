---
icon: shield-halved
---

# Introducción al SOC

Un **Security Operations Center (SOC)** es la instalación donde el equipo de seguridad de una organización monitoriza y analiza continuamente su postura de seguridad. El objetivo principal es **detectar, analizar y responder** a incidentes de ciberseguridad utilizando personas, procesos y tecnología de forma coordinada.

### Los tres pilares del SOC

Un SOC efectivo no funciona si falta alguno de estos tres elementos:

| Pilar          | Descripción                                                                                                           |
| -------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Personas**   | Analistas con formación técnica, mentalidad investigadora y conocimiento actualizado de las técnicas de los atacantes |
| **Procesos**   | Procedimientos estandarizados (playbooks), flujos claros de escalado, alineación con NIST, PCI-DSS, ISO 27001         |
| **Tecnología** | SIEM, EDR, Log Management, SOAR, Threat Intelligence, IDS/IPS                                                         |

### Tipos de modelo SOC

#### SOC Interno

La organización construye y gestiona su propio equipo de seguridad. Ofrece mayor control y respuesta más rápida, pero requiere un presupuesto elevado y sostenido. Típico en grandes bancos, operadoras de telecomunicaciones y agencias gubernamentales.

#### SOC Virtual

No tiene instalación permanente — el equipo trabaja en remoto desde distintas ubicaciones. Más flexible y económico, aunque la comunicación y coordinación requieren más esfuerzo.

#### SOC Cogestionado (MSSP)

Combina personal interno con un proveedor de servicios de seguridad gestionados externo (MSSP). El MSSP aporta herramientas, personal 24/7 y experiencia sectorial. Es el modelo más común en medianas empresas.

#### SOC de Comando

Supervisa múltiples SOC más pequeños dentro de una gran organización o región. Típico en multinacionales, grandes proveedores de telecomunicaciones y agencias de defensa.

### ¿Por qué existe el SOC?

Los ataques son constantes y automatizados — los bots prueban vulnerabilidades las 24 horas. Sin monitorización continua, una brecha puede pasar semanas sin detectarse. Además, la complejidad de los entornos modernos (endpoints, cloud, APIs, trabajo remoto...) hace imposible que un humano monitorice todo manualmente. El SIEM correlaciona millones de eventos y genera alertas priorizadas.

### Métricas clave

| Métrica                         | Descripción                                                                                      |
| ------------------------------- | ------------------------------------------------------------------------------------------------ |
| **MTTD** (Mean Time To Detect)  | Tiempo desde que ocurre el incidente hasta que se detecta. La media global actual es \~21 días.  |
| **MTTR** (Mean Time To Respond) | Tiempo desde la detección hasta la contención completa.                                          |
| **Tasa de Falsos Positivos**    | % de alertas que no son amenazas reales. Una tasa alta agota al equipo.                          |
| **Cobertura MITRE ATT\&CK**     | % de técnicas de ataque conocidas para las que hay detección activa. Los gaps son puntos ciegos. |

### El ciclo de vida de una alerta

Un evento ocurre en el sistema → se genera un log → el SIEM lo recibe y aplica reglas de correlación → si coincide con una regla, se genera una **alerta** → el analista L1 hace el **triaje** (¿FP o amenaza real?) → si es amenaza, se investiga en profundidad → se escala si es necesario → **respuesta** (contención, erradicación, recuperación) → documentación y mejora de las reglas.

### Herramientas principales

* **SIEM** — correlación de eventos y generación de alertas (Splunk, Microsoft Sentinel, QRadar, Wazuh)
* **EDR** — visibilidad y respuesta en endpoints (CrowdStrike, SentinelOne, Defender for Endpoint)
* **Log Management** — almacenamiento y búsqueda histórica (Elastic Stack)
* **SOAR** — automatización de la respuesta a incidentes
* **Threat Intelligence** — contexto sobre amenazas activas (VirusTotal, AbuseIPDB, MITRE ATT\&CK)

> &#x20;El SOC no es solo tecnología. Un SIEM con reglas perfectas operado por un equipo sin experiencia fallará igualmente. La combinación personas + procesos + tecnología es lo que hace funcionar un SOC de verdad.

> **Alert fatigue** (fatiga de alertas) es el mayor enemigo del SOC. Demasiados falsos positivos hacen que los analistas empiecen a ignorar alertas — y ahí es cuando los ataques reales pasan desapercibidos. Mantener la tasa de FP baja es tan importante como detectar amenazas.
