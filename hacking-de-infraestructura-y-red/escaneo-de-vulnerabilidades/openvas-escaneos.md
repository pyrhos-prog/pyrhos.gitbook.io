# OpenVAS — Escaneos

Para ejecutar un escaneo en GVM hay que crear los tres objetos en orden: **Credential** (si el escaneo es autenticado) → **Target** → **Task**. Una vez creada la Task, se lanza y se espera a que genere el Report.

### Paso 1 — Crear credenciales (escaneo autenticado)

Los escaneos con credenciales detectan muchas más vulnerabilidades que los escaneos de red simples. Para configurarlas antes de crear el Target:

Configuration → Credentials → New Credential

```
Name:           nombre descriptivo (ej. "Lab Linux SSH")
Type:           Username + Password / SSH Key / SMB / SNMP / ESXi
Allow insecure use: Off (recomendado)

Para SSH:
→ Username: usuario del sistema
→ Password: contraseña del sistema
→ O en lugar de contraseña: clave privada SSH en formato PEM

Para Windows (SMB):
→ Username: usuario con privilegios de administrador local
→ Password: contraseña del usuario
→ Domain: dominio Windows si aplica
```

### Paso 2 — Crear el Target

El Target define qué sistemas se van a escanear y con qué credenciales.

Configuration → Targets → New Target

```
Name:           nombre descriptivo del objetivo
Hosts:          IPs, rangos CIDR o nombres de host
                Ejemplos: 192.168.1.1
                          192.168.1.0/24
                          192.168.1.1-192.168.1.50
                          host.ejemplo.com

Exclude Hosts:  IPs a excluir dentro del rango (ej. impresoras, dispositivos frágiles)
Port List:      All IANA assigned TCP / All TCP / Top 100 TCP Ports (elegir según necesidad)

SSH Credentials:    seleccionar la credencial SSH creada en el paso anterior
SMB Credentials:    seleccionar la credencial SMB si es un entorno Windows
SNMP Credentials:   para dispositivos de red con SNMP configurado
```

### Paso 3 — Configuraciones de escaneo (Scan Configs)

Las Scan Configs determinan qué NVTs se ejecutan y la agresividad del escaneo. GVM incluye varias configuraciones predefinidas:

| Scan Config                     | Descripción                                                                      | Cuándo usarla                                  |
| ------------------------------- | -------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Full and Fast**               | Ejecuta todos los NVTs seguros de forma eficiente. La más usada.                 | Uso general, primer escaneo de un entorno      |
| **Full and Fast Ultimate**      | Como Full and Fast pero incluye NVTs que podrían interrumpir servicios           | Solo en entornos de lab, con permiso explícito |
| **Full and Very Deep**          | Más exhaustiva, tarda considerablemente más                                      | Cuando se necesita la cobertura máxima         |
| **Full and Very Deep Ultimate** | La más completa y más peligrosa para los sistemas                                | Solo en labs aislados                          |
| **System Discovery**            | Solo descubrimiento de hosts y servicios, sin comprobaciones de vulnerabilidades | Inventario inicial rápido                      |
| **Host Discovery**              | Solo comprueba qué hosts están activos                                           | Reconocimiento de red                          |
| **Empty**                       | Sin NVTs — base para crear configuraciones personalizadas                        | Configuraciones a medida                       |

Para la mayoría de casos, **Full and Fast** es la opción correcta.

### Paso 4 — Crear y lanzar la Task

Scans → Tasks → New Task

```
Name:           nombre descriptivo de la tarea
Scan Targets:   seleccionar el Target creado en el paso 2
Scanner:        OpenVAS Default (o un escáner remoto si está configurado)
Scan Config:    seleccionar la configuración (ej. Full and Fast)
Order hosts:    Sequential (en orden) / Random (aleatoriamente)
Max concurrently scanned hosts: número de hosts en paralelo
Max concurrently executed NVTs per host: NVTs paralelos por host
```

Una vez creada, lanzar la Task con el botón de play (triángulo verde). El progreso se muestra en la columna **Status** de la lista de Tasks:

```
Requested → Queued → Running (X%) → Done
```

### Monitorizar el progreso

Durante el escaneo se puede ver el progreso en tiempo real:

Scans → Tasks → clic en la Task activa

```
→ Progress bar con porcentaje de completitud
→ Hosts actualmente siendo escaneados
→ NVTs ejecutados
→ Tiempo transcurrido y estimación de finalización
```

Los escaneos completos de un rango de red pueden tardar desde minutos (Discovery) hasta horas (Full and Very Deep), dependiendo del número de hosts y la configuración de escaneo.

### Ver los resultados (Reports)

Al finalizar la Task, acceder a los resultados desde:

Scans → Reports → clic en el informe más reciente

#### Vista general del report

```
Resumen con gráficas:
→ Distribución de hallazgos por severidad (Critical/High/Medium/Low/Log)
→ Top 10 vulnerabilidades más severas
→ Top 10 hosts más vulnerables
→ Evolución respecto al escaneo anterior (si existe)
```

#### Vista por vulnerabilidades

La pestaña **Results** muestra todos los hallazgos con opciones de filtrado:

```
Columnas disponibles:
→ Severity (puntuación CVSS)
→ QoD (Quality of Detection) — confianza en el hallazgo, del 0% al 100%
→ Vulnerability — nombre del NVT que lo detectó
→ Host — IP del sistema afectado
→ Location — puerto/protocolo donde se detectó

Filtros disponibles:
→ Por severidad mínima
→ Por host específico
→ Por CVE ID
→ Por QoD mínimo
```

#### Quality of Detection (QoD)

El **QoD** es un concepto exclusivo de GVM que indica el nivel de confianza en que el hallazgo es un verdadero positivo:

| QoD  | Tipo de detección           | Fiabilidad                 |
| ---- | --------------------------- | -------------------------- |
| 100% | Exploit ejecutado con éxito | Certeza total              |
| 99%  | Remote active check         | Muy alta                   |
| 75%  | Remote banner check         | Media — depende del banner |
| 70%  | Remote version check        | Media                      |
| 50%  | Remote detection (general)  | Moderada                   |
| 30%  | Timeout / sin respuesta     | Baja                       |

Por defecto GSA filtra resultados con QoD < 70% para reducir falsos positivos. Se puede bajar el umbral en los filtros de resultados si se quiere ver todos los hallazgos.

#### Detalle de un hallazgo

Al hacer clic en un hallazgo individual se muestra:

```
Summary:        descripción breve de la vulnerabilidad
Insight:        explicación técnica de cómo funciona
Detection:      método usado para detectarla
Solution:       acción de remediación recomendada
References:     CVE IDs, CVSS vector, BIDs, links a advisories
```

> El filtro de **QoD ≥ 70%** predeterminado elimina la mayoría de falsos positivos automáticamente. Si los resultados parecen escasos, bajar el umbral a 50% para ver más hallazgos potenciales, pero verificarlos manualmente antes de reportarlos.

> Las Scan Configs con **Ultimate** en el nombre incluyen NVTs que pueden causar inestabilidad o interrupciones en servicios. Usarlas únicamente en entornos de laboratorio con sistemas que no sean de producción.
