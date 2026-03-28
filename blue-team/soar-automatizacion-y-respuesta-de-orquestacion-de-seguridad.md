---
icon: shield-halved
---

# SOAR — Automatización y Respuesta de Orquestación de Seguridad

**SOAR** (Security Orchestration, Automation and Response) es una plataforma que permite al SOC automatizar tareas repetitivas, orquestar herramientas de seguridad entre sí y estandarizar la respuesta a incidentes mediante **playbooks** automatizados.

### El problema que resuelve

**Sin SOAR** — analista recibe alerta de phishing:

El analista copia la URL manualmente, abre VirusTotal y la pega, espera el resultado, abre AbuseIPDB, busca la IP, va al Secure Email Gateway, abre el sistema de tickets y crea el caso manualmente, escribe el mensaje a Slack para avisar al equipo, y finalmente va al firewall a bloquear la IP. Tiempo total: **20-30 minutos por alerta**.

**Con SOAR** — misma alerta:

El SOAR recibe la alerta del SIEM automáticamente, consulta VirusTotal y AbuseIPDB en paralelo (2 segundos), enriquece la alerta con los resultados, si el score supera el umbral bloquea la IP en el firewall automáticamente, pone en cuarentena el email en todos los buzones, abre el ticket en Jira con todos los datos ya rellenos y notifica al canal de Slack del equipo. El analista recibe el ticket ya enriquecido con las acciones básicas tomadas. Tiempo total: **30 segundos automáticos**.

### Los tres pilares del SOAR

#### Orquestación

Conecta e integra todas las herramientas del SOC entre sí para que puedan intercambiar datos y ejecutar acciones:

```
SIEM ↔ EDR ↔ Firewall ↔ Threat Intel ↔ Ticketing ↔ Email ↔ AD
```

Sin orquestación, el analista es el "pegamento" entre herramientas. Con orquestación, las herramientas se hablan entre sí automáticamente.

#### Automatización

Ejecuta tareas repetitivas sin intervención humana: enriquecimiento de IOCs con APIs externas, acciones de respuesta (bloquear IP en firewall, aislar endpoint, deshabilitar cuenta en AD), apertura de tickets y envío de notificaciones.

#### Playbooks

Los playbooks son flujos de trabajo automatizados que definen exactamente qué pasos seguir para cada tipo de incidente. Garantizan que todos los analistas responden igual, que no se olvidan pasos en momentos de estrés, y que el tiempo de respuesta es predecible y medible.

### Ejemplo de playbook — Phishing

| Paso | Tipo     | Acción                                             |
| ---- | -------- | -------------------------------------------------- |
| 1    | AUTO     | Extraer la URL del cuerpo del email                |
| 2    | AUTO     | Consultar URL en VirusTotal API → obtener score    |
| 3    | AUTO     | Consultar URL en URLScan.io → capturar pantalla    |
| 4    | AUTO     | Extraer la IP del dominio y consultar en AbuseIPDB |
| 5    | DECISIÓN | ¿Score > 70?                                       |
| 5a   | AUTO     | Bloquear URL en el proxy web                       |
| 5b   | AUTO     | Cuarentena del email en todos los buzones          |
| 5c   | AUTO     | Crear ticket P2 en Jira con toda la info           |
| 5d   | AUTO     | Notificar en #soc-alerts en Slack                  |
| 5e   | MANUAL   | Asignar alerta a analista para confirmación        |
| 6    | AUTO     | Documentar todas las acciones en el ticket         |

### Tipos de acciones en un playbook

* **Acciones automáticas** — consultas a APIs, bloqueos en firewall, apertura de tickets, notificaciones, búsquedas en SIEM
* **Puntos de aprobación manual** — acciones irreversibles o de alto impacto requieren aprobación antes de ejecutarse (deshabilitar cuenta, eliminar archivo, aislar servidor crítico)
* **Acciones condicionales** — si score > X → acción A; si sistema crítico → escalar; si fuera del horario → notificar al de guardia

> Un playbook mal diseñado puede ser peor que no tener SOAR. Acciones automáticas incorrectas (bloquear IPs legítimas, deshabilitar cuentas de servicio críticas) pueden causar interrupciones del negocio. Los playbooks deben probarse exhaustivamente antes de activar las acciones automáticas irreversibles.

### Impacto en el trabajo del analista

| Sin SOAR                                                  | Con SOAR                                                 |
| --------------------------------------------------------- | -------------------------------------------------------- |
| Trabajo repetitivo: copiar IOCs, buscar en múltiples webs | Alertas ya enriquecidas con contexto al recibirlas       |
| Cada analista tiene su propio proceso                     | Respuesta uniforme, independiente del analista           |
| Tiempo de respuesta variable                              | MTTR predecible y medible                                |
| Documentación manual                                      | Documentación automática de todo lo que hace el playbook |

El analista sigue haciendo lo que requiere juicio: validar los resultados automáticos, investigar los casos que superan la lógica del playbook y mejorar los playbooks basándose en la experiencia.

### Soluciones SOAR principales

| Solución                         | Características                                                 |
| -------------------------------- | --------------------------------------------------------------- |
| **Splunk SOAR**                  | Líder del mercado, 300+ integraciones, playbooks en Python      |
| **Microsoft Sentinel Playbooks** | Integrado en Sentinel via Azure Logic Apps, sin coste adicional |
| **Palo Alto XSOAR**              | Muy completo, orientado a grandes SOCs                          |
| **TheHive + Cortex**             | 100% open source, gratuito, gran comunidad de analyzers         |

> **TheHive + Cortex** es la combinación perfecta para aprender SOAR sin coste. Cortex tiene docenas de analyzers gratuitos que consultan VirusTotal, AbuseIPDB, Shodan y MISP automáticamente, demostrando exactamente lo que hace un SOAR comercial.
