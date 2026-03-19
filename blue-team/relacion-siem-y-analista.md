---
icon: shield-halved
---

# Relación SIEM y Analista

### ¿Qué es el SIEM?

SIEM (Security Information and Event Management) es la solución central del SOC. Combina la recopilación de información de seguridad con la gestión de eventos en tiempo real. Su objetivo final es detectar amenazas a través del registro y correlación de eventos.

```
SIEM = SIM + SEM

SIM (Security Information Management):
→ Recopilación y almacenamiento de logs a largo plazo
→ Generación de informes de cumplimiento normativo

SEM (Security Event Management):
→ Monitorización en tiempo real de eventos de seguridad
→ Correlación de eventos y generación de alertas inmediatas
```

### Cómo funciona el SIEM

```
Fuentes de datos (logs) → SIEM → Correlación → Alertas → Analista

1. RECOPILACIÓN
   El SIEM recibe logs de toda la infraestructura:
   → Endpoints (Windows Event Logs, EDR)
   → Servidores (aplicaciones, bases de datos)
   → Red (firewall, router, switch, IDS/IPS)
   → Cloud (AWS CloudTrail, Azure Monitor)
   → Identidad (Active Directory, LDAP)
   → Seguridad (antivirus, proxy, VPN)

2. NORMALIZACIÓN
   Los logs llegan en formatos distintos
   El SIEM los convierte a un formato unificado para poder correlacionarlos

3. CORRELACIÓN
   Aplica reglas para detectar patrones sospechosos
   → "Si el mismo usuario falla 20 logins en 10 segundos → alerta"
   → "Si una IP del firewall también aparece en el IDS → alerta"

4. ALERTA GENERADA
   El analista la recibe en el panel del SIEM
   → Canal principal: alertas disponibles para cualquier analista
   → Canal de investigación: alertas que ya tiene alguien trabajando
```

### El SIEM y el analista — cómo trabajan juntos

#### El rol del analista frente al SIEM

```
El SIEM genera alertas. El analista las investiga.

Aunque el SIEM tiene muchas características avanzadas
(dashboards, correlación, búsqueda de logs...),
el trabajo diario del analista L1 se centra en:

1. Monitorizar el panel de alertas
   → Revisar el canal principal del SIEM continuamente
   → Identificar alertas nuevas y priorizarlas

2. Tomar posesión de una alerta
   → Hacer clic en "Tomar propiedad" / "Take Ownership"
   → La alerta pasa al canal de investigación personal
   → Los compañeros ven que esa alerta ya está siendo trabajada
   → Evita que dos analistas investiguen el mismo caso

3. Revisar los detalles de la alerta
   → El SIEM muestra la información básica: IP, hostname, usuario, hash...
   → Con esa información el analista va a las otras herramientas (EDR, logs, TI)
   → El SIEM es el punto de partida, no el único lugar donde se investiga

4. Concluir y documentar
   → Registrar la conclusión en el SIEM
   → FP con justificación o escalar como amenaza real
```

#### Quién configura el SIEM (no el analista L1)

```
El analista L1 normalmente NO toca la configuración del SIEM.
Hay otros roles para eso:

Ingeniero de Seguridad SOC:
→ Crea y ajusta las reglas de correlación
→ Integra nuevas fuentes de logs
→ Configura los dashboards
→ Optimiza el rendimiento

Analista L3 / Threat Hunter:
→ Crea nuevas reglas de detección basadas en TTPs
→ Ajusta las reglas para reducir falsos positivos
→ Desarrolla consultas de hunting en el SIEM

El analista L1 puede y debe:
→ Reportar cuando una regla genera demasiados FPs
→ Sugerir ajustes basados en los patrones que observa
→ Contribuir con feedback al equipo que gestiona las reglas
```

### Reglas del SIEM — cómo se generan las alertas

Las reglas definen cuándo se genera una alerta. Son filtros que el SIEM aplica sobre los eventos que recibe.

```
Ejemplo básico — Bruteforce de contraseña:

Sin regla: el SIEM recibe todos los EventID 4625 (login fallido)
           sin generar ninguna alerta. Solo almacena el log.

Con regla: "Si el EventID 4625 ocurre más de 20 veces
            desde la misma IP en menos de 10 segundos
            → generar alerta de severidad Media"

Resultado: el analista recibe una alerta con todos los datos
           del intento de bruteforce para investigar
```

#### Tipos de reglas

```
Umbral (Threshold):
→ Más de N eventos en Y segundos → alerta
→ Ejemplo: 20 logins fallidos en 10s → bruteforce

Secuencia:
→ Evento A seguido de Evento B → alerta
→ Ejemplo: escaneo de puertos + intento de conexión → reconocimiento activo

Correlación entre fuentes:
→ La misma IP aparece en firewall Y en IDS → alerta
→ Mayor precisión al cruzar datos de distintos sistemas

Anomalía (UEBA):
→ El usuario se comporta de forma distinta a su baseline
→ Ejemplo: empleado de oficina conectándose a las 3am

Lista negra (Blacklist):
→ Cualquier comunicación con IOCs conocidos → alerta
→ Basado en feeds de Threat Intelligence
```

### Falsos positivos — el mayor reto del SIEM

Una de las responsabilidades más importantes del analista es **identificar los falsos positivos** y contribuir a que no se repitan.

```
¿Qué es un falso positivo (FP)?
→ Una alerta generada por el SIEM que no corresponde
  a ninguna amenaza real

¿Por qué ocurren?
→ Las reglas son demasiado genéricas
→ No se han contemplado excepciones para actividades legítimas
→ La infraestructura ha cambiado y las reglas no se han actualizado

Ejemplo del material de estudio (LetsDefend):
Regla: alerta cuando una URL contiene la palabra "union"
       (intentando detectar SQL Injection)

FP generado: un usuario busca en Google:
             "https://www.google.com/search?q=sql+union+usage"
             → La URL contiene "union" → alerta generada
             → Pero no hay inyección SQL real

Cómo manejarlo:
→ El analista determina que es un FP
→ Documenta por qué es un FP (búsqueda legítima en Google)
→ Reporta al equipo de ingeniería SIEM
→ El ingeniero ajusta la regla:
  "Alertar solo si 'union' aparece en una petición POST
   a los endpoints de la aplicación, no en URLs de Google"
```

#### Proceso para reportar un FP recurrente

```
1. Identificar el patrón
   → ¿Qué regla lo genera?
   → ¿En qué contexto exacto se produce?
   → ¿Cuántas veces al día/semana?

2. Documentar el caso
   → Ejemplos concretos de los FPs con sus datos
   → Explicación de por qué no es una amenaza real
   → Propuesta de cómo ajustar la regla

3. Comunicar al equipo de ingeniería SIEM
   → Con el contexto completo
   → Sin modificar la regla por cuenta propia

4. Verificar después del ajuste
   → ¿Bajó la tasa de FPs?
   → ¿La regla sigue detectando amenazas reales?
```

### Soluciones SIEM comunes

```
Microsoft Sentinel:
→ Cloud-native, integrado en Azure
→ Lenguaje de consulta: KQL (Kusto Query Language)
→ Muy común en entornos Microsoft (M365, Azure AD, Defender)

Splunk:
→ Muy potente para búsquedas complejas
→ Lenguaje: SPL (Search Processing Language)
→ Gran ecosistema de apps y add-ons

IBM QRadar:
→ Clásico en sector enterprise y financiero
→ Motor de correlación muy maduro

Elastic SIEM (Kibana):
→ Open source en la versión básica
→ Muy flexible y escalable

Wazuh (open source, gratuito):
→ Ideal para aprender y para entornos con presupuesto limitado
→ Integración con MITRE ATT&CK
→ Muy usado en laboratorios y pequeñas organizaciones
```

> El SIEM es el punto de partida de la investigación, no el único lugar donde se investiga. Los datos del SIEM llevan al analista a consultar el EDR, los logs, la Threat Intelligence... El SIEM muestra que algo pasó — el analista descubre qué pasó realmente.

> La calidad de un SIEM depende directamente de la calidad de sus reglas. Un SIEM con reglas mal configuradas genera alertas constantemente (fatiga de alertas) o no detecta amenazas reales. El feedback del analista al equipo de ingeniería es fundamental para mejorar las reglas.

> Cuando el analista identifica un falso positivo recurrente, comunicarlo siempre al equipo en lugar de simplemente cerrarlo. Un FP no reportado seguirá generándose indefinidamente, consumiendo tiempo del equipo y contribuyendo a la fatiga de alertas.
