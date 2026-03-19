---
icon: shield-halved
---

# SOAR — Automatización y Respuesta de Orquestación de Seguridad

**SOAR (Security Orchestration, Automation and Response)** es una plataforma que permite al SOC **automatizar tareas repetitivas**, **orquestar herramientas** de seguridad entre sí, y **estandarizar la respuesta** a incidentes mediante playbooks automatizados.

```
El problema que resuelve el SOAR:

Sin SOAR — analista recibe alerta de phishing:
→ Copia la URL manualmente
→ Abre VirusTotal y la pega → espera resultado
→ Abre AbuseIPDB y busca la IP → espera resultado
→ Va al Secure Email Gateway y busca el email
→ Abre el sistema de tickets y crea manualmente el caso
→ Escribe un mensaje a Slack para avisar al equipo
→ Va al firewall y bloquea la IP manualmente
Tiempo total: 20-30 minutos por alerta

Con SOAR — misma alerta:
→ El SOAR recibe la alerta del SIEM automáticamente
→ Consulta VirusTotal, AbuseIPDB y URLScan en paralelo (2 segundos)
→ Enriquece la alerta con todos los resultados
→ Si el score supera el umbral: bloquea la IP en el firewall automáticamente
→ Pone en cuarentena el email en todos los buzones
→ Abre el ticket en Jira con todos los datos ya rellenos
→ Notifica al canal de Slack del equipo
→ El analista recibe el ticket ya enriquecido y las acciones básicas tomadas
Tiempo total: 30 segundos automáticos + el analista solo revisa y decide
```

### Los tres pilares del SOAR

#### Orquestación

```
Conecta e integra todas las herramientas del SOC entre sí
para que puedan intercambiar datos y ejecutar acciones:

SIEM ↔ EDR ↔ Firewall ↔ Threat Intel ↔ Ticketing ↔ Email ↔ AD

Sin orquestación: el analista es el "pegamento" entre herramientas
Con orquestación: las herramientas se hablan entre sí automáticamente

Ejemplos de integración:
→ SIEM detecta IP maliciosa → SOAR la bloquea en el firewall
→ EDR detecta malware → SOAR aísla el endpoint automáticamente
→ SIEM detecta cuenta comprometida → SOAR deshabilita la cuenta en AD
→ Alerta de phishing → SOAR pone en cuarentena el email en todos los buzones
```

#### Automatización

```
Ejecuta tareas repetitivas sin intervención humana:

Enriquecimiento automático:
→ Consultar IOCs en VirusTotal, AbuseIPDB, Shodan, MISP...
→ Obtener información WHOIS del dominio
→ Geolocalizar las IPs involucradas
→ Buscar el hash en bases de datos de malware
→ Todo antes de que el analista abra la alerta

Acciones automáticas de respuesta:
→ Bloquear IP en firewall si score > X
→ Aislar endpoint si EDR detecta actividad crítica
→ Deshabilitar cuenta si hay indicios de compromiso
→ Poner email en cuarentena si contiene IOC conocido

Notificaciones automáticas:
→ Abrir ticket en Jira/ServiceNow con datos pre-rellenos
→ Notificar al canal de Slack/Teams del equipo
→ Enviar resumen por email al responsable de turno
```

#### Respuesta estandarizada (Playbooks)

```
Los playbooks son flujos de trabajo automatizados que definen
exactamente qué pasos seguir para cada tipo de incidente.

Beneficios:
→ Todos los analistas responden igual al mismo tipo de incidente
→ No se olvidan pasos en momentos de estrés
→ Menor dependencia de la experiencia individual
→ Tiempo de respuesta predecible y medible
→ Documentación automática de todas las acciones tomadas
```

### Playbooks — cómo funcionan

Un playbook es una secuencia de pasos que puede combinar acciones automáticas y puntos de decisión manual.

#### Ejemplo — Playbook de phishing

```
TRIGGER: Alerta del SIEM "Email con URL maliciosa detectada"

PASO 1 [AUTO] → Extraer la URL del email del cuerpo del mensaje

PASO 2 [AUTO] → Consultar URL en VirusTotal API
                → Obtener score de detección (0-100)

PASO 3 [AUTO] → Consultar URL en URLScan.io
                → Obtener captura de pantalla de la web

PASO 4 [AUTO] → Extraer la IP del dominio de la URL
                → Consultar IP en AbuseIPDB

PASO 5 [DECISIÓN] → ¿Score VirusTotal > 70?
  SÍ →
    PASO 5a [AUTO] → Bloquear URL en el proxy web
    PASO 5b [AUTO] → Poner email en cuarentena en TODOS los buzones
    PASO 5c [AUTO] → Crear ticket P2 en Jira con toda la info
    PASO 5d [AUTO] → Notificar en #soc-alerts en Slack
    PASO 5e [MANUAL] → Asignar alerta a analista para confirmación
  NO →
    PASO 5f [MANUAL] → Asignar a analista L1 para revisión manual

PASO 6 [AUTO] → Documentar todas las acciones en el ticket
PASO 7 [AUTO] → Actualizar métricas del SOC (tiempo de respuesta)
```

#### Ejemplo — Playbook de endpoint comprometido

```
TRIGGER: EDR reporta actividad crítica en endpoint (malware confirmado)

PASO 1 [AUTO] → Obtener información del endpoint: usuario, dept., criticidad
PASO 2 [AUTO] → Aislar el endpoint en el EDR (cortar red)
PASO 3 [AUTO] → Capturar snapshot del estado del sistema para forense
PASO 4 [AUTO] → Extraer los IOCs del EDR (hash, IP C2, dominio)
PASO 5 [AUTO] → Buscar los IOCs en todos los demás endpoints (hunting)
PASO 6 [AUTO] → Crear ticket P1/P2 según criticidad del sistema
PASO 7 [AUTO] → Notificar al responsable del equipo de IR
PASO 8 [AUTO] → Si sistema crítico (DC, fileserver): alertar CISO automáticamente
PASO 9 [MANUAL] → IR toma el control de la investigación
```

#### Ejemplo — Playbook de bruteforce exitoso (login tras varios fallos)

```
TRIGGER: SIEM detecta login exitoso precedido de 50 fallos desde misma IP

PASO 1 [AUTO] → Obtener datos de la cuenta comprometida
PASO 2 [AUTO] → Verificar si la IP está en whitelist de IPs autorizadas
  SÍ → Cerrar como FP, documentar
  NO →
    PASO 3 [AUTO] → Geolocalizar la IP → ¿país inusual para este usuario?
    PASO 4 [AUTO] → Comprobar si hay sesión activa simultánea desde otra IP
    PASO 5 [MANUAL] → Contactar al usuario (email/teléfono) para verificar
      Usuario confirma que es él → cerrar, añadir IP a whitelist temporal
      Usuario NO reconoce el acceso →
        PASO 6 [AUTO] → Deshabilitar cuenta en Active Directory
        PASO 7 [AUTO] → Invalidar todas las sesiones activas
        PASO 8 [AUTO] → Escalar a L2 para investigación de cuenta comprometida
        PASO 9 [AUTO] → Crear ticket P2
```

### Tipos de acciones en un playbook

```
Acciones automáticas (no requieren intervención humana):
→ Consultas a APIs externas (VirusTotal, AbuseIPDB, Shodan)
→ Acciones en herramientas internas (bloquear en firewall, aislar en EDR)
→ Crear y actualizar tickets
→ Enviar notificaciones
→ Buscar en logs del SIEM/Elastic
→ Ejecutar scripts de recopilación de datos

Puntos de aprobación manual (human-in-the-loop):
→ Acciones irreversibles o de alto impacto requieren aprobación
→ Ejemplos: deshabilitar cuenta, eliminar archivo, aislar servidor crítico
→ El SOAR espera aprobación del analista antes de continuar
→ Si no hay respuesta en N minutos → escalar o ejecutar acción por defecto

Acciones condicionales:
→ Si score > X → acción A
→ Si sistema crítico → escalar automáticamente
→ Si fuera del horario laboral → notificar al responsable de guardia
```

### SOAR y el analista — impacto en el trabajo diario

```
Lo que cambia para el analista con SOAR:

Antes del SOAR (sin automatización):
→ El analista hace trabajo repetitivo: copiar IOCs, buscar en múltiples webs...
→ Cada analista tiene su propio proceso de investigación
→ Las respuestas varían en calidad según la experiencia
→ El tiempo de respuesta depende de la velocidad del analista

Con SOAR:
→ El analista recibe alertas ya enriquecidas con contexto
→ Las acciones básicas de respuesta ya están tomadas
→ El analista puede centrarse en lo que requiere juicio humano
→ La calidad de respuesta es uniforme independientemente del analista
→ El tiempo de respuesta es predecible y medible

Lo que el analista gana con SOAR:
→ Menos tareas mecánicas y repetitivas
→ Más tiempo para análisis de calidad
→ Contexto completo ya disponible al abrir la alerta
→ Documentación automática de todo lo que hace el playbook

Lo que el analista sigue haciendo:
→ Tomar decisiones complejas que requieren juicio
→ Validar los resultados automáticos
→ Investigar los casos que superan la lógica del playbook
→ Mejorar los playbooks basándose en la experiencia
```

### Soluciones SOAR principales

```
Splunk SOAR (antes Phantom):
→ Líder del mercado
→ Más de 300 integraciones nativas (apps para cada herramienta)
→ Playbooks con Python
→ Muy potente pero requiere licencia costosa

Microsoft Sentinel Playbooks:
→ Integrado en Microsoft Sentinel via Azure Logic Apps
→ Interfaz visual (drag & drop) o código
→ Sin coste adicional para clientes de Sentinel
→ Ideal para entornos Microsoft

Palo Alto XSOAR (antes Demisto):
→ Muy completo, orientado a grandes SOCs
→ Casos de uso muy documentados

TheHive + Cortex (open source):
→ TheHive: gestión de casos e investigación
→ Cortex: motor de automatización que ejecuta "analyzers" y "responders"
→ 100% open source y gratuito
→ Gran comunidad de analyzers: VirusTotal, AbuseIPDB, Shodan, MISP...
→ Ideal para aprender SOAR y para SOCs con presupuesto limitado

n8n / Shuffle (open source):
→ Plataformas de automatización de workflows
→ Menos específicas para seguridad que las anteriores
→ Pero más accesibles y gratuitas para empezar
```

### Métricas de un SOAR efectivo

```
¿Cómo medir si el SOAR está aportando valor?

Tiempo de respuesta (MTTR):
→ ¿Cuánto tiempo tardaba el equipo antes del SOAR?
→ ¿Cuánto tarda ahora con el SOAR?
→ Reducción del 60-80% en el MTTR es esperable con SOAR bien implementado

Alertas gestionadas por analista:
→ Con SOAR se pueden gestionar 3-5x más alertas por analista
→ Porque el trabajo repetitivo lo hace el SOAR

Consistencia de respuesta:
→ % de incidentes gestionados siguiendo el playbook vs ad-hoc
→ Alta consistencia = menor riesgo de pasos olvidados

Cobertura de playbooks:
→ % de tipos de alertas con playbook definido
→ Las alertas sin playbook son candidatas a crear uno nuevo
```

> El SOAR no reemplaza al analista — lo potencia. Las decisiones complejas que requieren juicio humano siguen siendo del analista. El SOAR elimina el trabajo mecánico para que el analista pueda centrarse en lo que realmente importa.

> TheHive + Cortex es la combinación perfecta para aprender SOAR sin coste. Cortex tiene docenas de analyzers gratuitos que consultan VirusTotal, AbuseIPDB, Shodan y MISP automáticamente, demostrando exactamente lo que hace un SOAR comercial.

> Un playbook mal diseñado puede ser peor que no tener SOAR: acciones automáticas incorrectas (bloquear IPs legítimas, deshabilitar cuentas de servicio críticas) pueden causar interrupciones del negocio. Los playbooks deben probarse exhaustivamente antes de activar las acciones automáticas irreversibles.
