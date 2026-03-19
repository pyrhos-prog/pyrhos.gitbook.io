---
icon: shield-halved
---

# Feed de Inteligencia sobre Amenazas (Threat Intelligence)

### ¿Qué es la Threat Intelligence?

**Threat Intelligence (TI)** es el conocimiento basado en evidencias sobre amenazas existentes o emergentes que ayuda a tomar decisiones de seguridad informadas. Va más allá de los datos brutos (logs, alertas) para proporcionar **contexto, mecanismos, indicadores e implicaciones** sobre las amenazas.

```
Dato vs Información vs Inteligencia:

Dato:    "45.33.32.156"
         (una IP, sin contexto)

Información: "45.33.32.156 tiene 87/100 en AbuseIPDB"
             (contexto básico de reputación)

Inteligencia: "45.33.32.156 es el C2 principal del grupo APT28 (Fancy Bear),
               vinculado al gobierno ruso, que está actualmente atacando
               organizaciones del sector energético europeo usando phishing
               con documentos Word que explotan CVE-2023-XXXX"
              (contexto completo para tomar decisiones)
```

### Tipos de Threat Intelligence

#### Estratégica

```
Para quién: CISO, directores, comité de seguridad
Nivel: alto nivel, sin detalles técnicos

Qué incluye:
→ Tendencias de amenazas por sector (energía, finanzas, salud...)
→ Grupos APT activos y sus objetivos habituales
→ Riesgos geopolíticos que afectan a la ciberseguridad
→ Impacto financiero de los tipos de ataque más frecuentes
→ Predicciones de amenazas futuras

Ejemplo:
"Los grupos de ransomware están aumentando sus ataques al sector sanitario
español un 40% respecto al año anterior, con un rescate medio de 800.000€"

Uso: justificar inversiones en seguridad, priorizar áreas de mejora
```

#### Táctica

```
Para quién: equipo técnico, arquitectos de seguridad, ingenieros SOC
Nivel: TTPs (Tactics, Techniques and Procedures)

Qué incluye:
→ Técnicas específicas que usan los grupos de amenaza
→ Herramientas que usa el atacante (Cobalt Strike, Metasploit, Mimikatz...)
→ Fases del ataque: acceso inicial, persistencia, movimiento lateral...
→ Mapeado a MITRE ATT&CK

Ejemplo:
"APT29 (Cozy Bear) está usando spearphishing con documentos PDF
que ejecutan PowerShell vía macros para el acceso inicial,
seguido de Cobalt Strike como framework de C2 usando HTTPS
con certificados Let's Encrypt para evadir inspección"

Uso: ajustar reglas de detección del SIEM, mejorar playbooks,
     configurar controles defensivos específicos
```

#### Operacional

```
Para quién: analistas SOC, respondedores de incidentes
Nivel: campañas y ataques activos en curso

Qué incluye:
→ Campañas de ataque activas en este momento
→ Indicadores específicos de ataques en curso
→ Advertencias de amenazas inminentes contra sectores específicos
→ Información sobre nuevas vulnerabilidades siendo explotadas activamente

Ejemplo:
"Campaña activa de phishing en España usando emails que simulan ser
de la AEAT (Agencia Tributaria). Los emails tienen adjunto 'declaracion_2024.pdf'
que en realidad descarga un troyano bancario"

Uso: buscar emails similares en la organización, alertar a usuarios,
     bloquear IOCs relacionados
```

#### Técnica

```
Para quién: analistas SOC, herramientas de seguridad
Nivel: IOCs (Indicators of Compromise) concretos

Qué incluye:
→ Hashes de archivos maliciosos (MD5, SHA1, SHA256)
→ IPs de servidores de C2 y distribución de malware
→ Dominios maliciosos
→ URLs de descarga de payloads
→ Patrones de red (User-Agents, headers, beacon intervals)
→ Firmas YARA y reglas Sigma

Uso: integrar directamente en SIEM, EDR, firewall, proxy
     para detección automática
```

### IOCs — Indicadores de Compromiso

Los IOCs son los artefactos técnicos que indican que un sistema puede estar comprometido.

```
Tipos de IOCs:

Basados en archivo:
→ Hashes: MD5, SHA1, SHA256 del archivo malicioso
   MD5:    d41d8cd98f00b204e9800998ecf8427e
   SHA256: e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
→ Nombre del archivo: svchost32.exe, update.ps1
→ Ruta del archivo: C:\Users\user\AppData\Roaming\malware.exe
→ Tamaño del archivo
→ Strings específicas dentro del archivo

Basados en red:
→ IPs de C2: 185.220.101.47
→ Dominios de C2: evil-c2-domain.com
→ URLs específicas: http://evil.com/payload/update.exe
→ User-Agent strings específicos del malware
→ Patrones de beaconing (intervalo, tamaño de paquetes)
→ Certificados TLS específicos (hash del certificado)
→ JA3/JA3S hashes (fingerprint de conexiones TLS)

Basados en host:
→ Claves de registro creadas por el malware
→ Nombres de servicios instalados
→ Nombres de mutex (identificador único del malware en memoria)
→ Nombres de procesos o servicios
→ Comandos o scripts ejecutados
```

#### Pirámide del Dolor

La pirámide del dolor (Pyramid of Pain) muestra el impacto que tiene para el atacante que el defensor detecte y bloquee cada tipo de IOC.

```
                    TTPs               ← Muy difícil de cambiar para el atacante
                 ___________
                Herramientas           ← Difícil (requiere modificar el malware)
              _______________
           Artifacts de red/host       ← Molesto (tiempo de re-configuración)
         ___________________
              Dominios                 ← Molesto (registrar nuevo dominio)
           ___________
                IPs                   ← Fácil (cambiar de servidor en minutos)
             _______
              Hashes                  ← Trivial (recompilar el malware cambia el hash)
```

```
Implicación práctica:
→ Bloquear un hash → el atacante lo cambia en minutos (recompilar)
→ Bloquear una IP → el atacante cambia de servidor en horas
→ Detectar una TTP → el atacante tiene que cambiar su metodología completa
                      → mucho más costoso y difícil

Por eso, las mejores reglas de detección se basan en TTPs,
no solo en IOCs específicos (hashes e IPs)
```

### Fuentes de Threat Intelligence

#### Fuentes gratuitas

```
Reputación y análisis de IOCs:

VirusTotal (https://virustotal.com):
→ Análisis con 70+ motores AV de archivos, URLs, IPs, dominios
→ Historial de detecciones
→ Relaciones entre muestras, dominios e IPs
→ Gráfico de relaciones entre IOCs
→ API gratuita con límites

AbuseIPDB (https://abuseipdb.com):
→ Base de datos comunitaria de IPs maliciosas
→ Score de 0-100 basado en reportes de la comunidad
→ Historial de actividad maliciosa de cada IP
→ Categorías: SSH bruteforce, web attacks, spam, C2...
→ API gratuita

URLScan.io (https://urlscan.io):
→ Escanea y analiza URLs en sandbox
→ Captura de pantalla de la web
→ Tecnologías detectadas, certificados, IPs
→ Excelente para analizar URLs de phishing

Shodan (https://shodan.io):
→ Motor de búsqueda de dispositivos conectados a internet
→ Información sobre IPs: puertos abiertos, servicios, banners
→ Muy útil para enriquecer IPs durante la investigación
→ ¿Es un servidor web legítimo o un C2 en un VPS?

GreyNoise (https://greynoise.io):
→ Diferencia entre ruido de internet y amenazas reales
→ "Esta IP es un escáner masivo de internet, no un atacante dirigido"
→ Reduce FPs relacionados con escaneos generales de internet

AlienVault OTX (https://otx.alienvault.com):
→ Plataforma colaborativa de Threat Intelligence
→ Pulses: colecciones de IOCs sobre una amenaza o campaña
→ Comunidad activa que comparte indicadores

Bases de datos de malware:
MalwareBazaar (https://bazaar.abuse.ch):
→ Repositorio de muestras de malware
→ Buscar por hash, familia de malware, etiquetas

URLhaus (https://urlhaus.abuse.ch):
→ URLs de distribución de malware
→ Buscar dominios e IPs asociadas a distribución de malware

ThreatFox (https://threatfox.abuse.ch):
→ IOCs de malware activo: IPs de C2, dominios, URLs, hashes
→ Incluye contexto: qué familia de malware, cuándo se detectó

MITRE ATT&CK (https://attack.mitre.org):
→ Framework de TTPs de grupos APT
→ Perfiles de grupos de amenaza con sus técnicas específicas
→ Base de conocimiento fundamental para cualquier analista SOC
```

#### Fuentes de pago (para referencia)

```
Recorded Future:
→ Inteligencia predictiva avanzada
→ Alertas tempranas sobre amenazas emergentes
→ Muy usado en grandes corporaciones y sector financiero

Mandiant Threat Intelligence (Google):
→ Investigación de APTs de primera línea
→ Informes detallados de campañas activas

CrowdStrike Falcon Intelligence:
→ Integrado en el EDR CrowdStrike
→ Inteligencia sobre más de 200 grupos de adversarios rastreados

IBM X-Force:
→ Base de datos masiva de vulnerabilidades y IOCs
→ Integrada con QRadar

Palo Alto Unit42:
→ Investigación de amenazas del equipo de inteligencia de Palo Alto
→ Informes públicos gratuitos de gran calidad
```

### Cómo usa el analista la Threat Intelligence

#### Durante el triaje e investigación

```
Flujo típico cuando el analista tiene una IP sospechosa:

1. AbuseIPDB → ¿tiene reportes? ¿de qué tipo?
   Score 0  → probablemente legítima, investigar más
   Score 87 → reportada múltiples veces por SSH bruteforce

2. VirusTotal → ¿algún AV la detecta como maliciosa?
   → ¿Con qué dominios o hashes se asocia?

3. Shodan → ¿qué servicios expone esa IP?
   → Puerto 22, 80, 443 y apariencia de hosting → VPS de atacante
   → Cloudflare CDN con certificado válido → probablemente legítima

4. GreyNoise → ¿es ruido de internet o amenaza dirigida?
   "This IP is commonly seen scanning the internet" → reduce urgencia
   "This IP is not commonly seen" → más sospechoso

5. MITRE ATT&CK → si se conoce el grupo o la técnica:
   → ¿Qué otras técnicas usan habitualmente?
   → ¿Qué más debería buscar en el entorno?

Tiempo de consulta con herramientas y API del SOAR: 10-15 segundos
Sin SOAR (manual): 5-10 minutos
```

#### Integración en el SIEM

```
Los feeds de TI se integran directamente en el SIEM para
detección automática:

→ Importar listas de IPs/dominios maliciosos conocidos
→ El SIEM alerta automáticamente cuando cualquier sistema
   se comunica con esos IOCs
→ Los feeds se actualizan diariamente o en tiempo real

Formatos estándar para importar feeds:
→ STIX (Structured Threat Information eXpression):
  Formato XML/JSON para describir amenazas e IOCs

→ TAXII (Trusted Automated eXchange of Indicator Information):
  Protocolo para compartir feeds de TI entre herramientas

→ CSV / JSON simple:
  Muchas fuentes gratuitas ofrecen listas en CSV o JSON
```

#### Hunting proactivo con TI

```
La TI permite ir más allá de esperar alertas:

Ejemplo de uso en hunting:
1. Noticia: "Grupo APT29 usando dominios con patrón *.azurewebsites-cdn.com"
2. Analista busca en los logs DNS de los últimos 30 días:
   *.azurewebsites-cdn.com → ¿algún sistema de la organización hizo esa consulta?
3. Si hay coincidencias → investigar esos sistemas
4. Si no hay → crear regla en el SIEM para alertar si aparece en el futuro

Esto es threat hunting basado en inteligencia táctica.
```

### MITRE ATT\&CK — el framework fundamental

```
MITRE ATT&CK es una base de conocimiento de las tácticas y técnicas
que usan los atacantes reales, organizada como una matriz.

Tácticas (el "qué quieren lograr"):
TA0001 Reconocimiento
TA0002 Desarrollo de recursos
TA0003 Acceso inicial
TA0004 Ejecución
TA0005 Persistencia
TA0006 Escalada de privilegios
TA0007 Evasión de defensas
TA0008 Acceso a credenciales
TA0009 Descubrimiento
TA0010 Movimiento lateral
TA0011 Recopilación
TA0012 Comando y control (C2)
TA0013 Exfiltración
TA0014 Impacto

Técnicas (el "cómo lo hacen"):
T1059.001 → PowerShell (subtécnica de Command and Scripting Interpreter)
T1003.001 → LSASS Memory (subtécnica de OS Credential Dumping)
T1078     → Valid Accounts (usar credenciales robadas)
T1053.005 → Scheduled Task (persistencia vía tarea programada)
T1486     → Data Encrypted for Impact (ransomware)

Uso práctico en el SOC:
→ Cuando el analista investiga una técnica, ATT&CK explica:
  ¿Qué es? ¿Cómo se detecta? ¿Qué mitigaciones existen?
  ¿Qué grupos la usan? ¿Con qué otras técnicas se combina?

→ ATT&CK Navigator: visualizar la cobertura de detección del SOC
  → https://mitre-attack.github.io/attack-navigator/
  → Verde = tenemos detección, Rojo = punto ciego
```

> La **pirámide del dolor** es uno de los conceptos más importantes de Threat Intelligence: bloquear IPs y hashes es fácil para el atacante de evadir, pero detectar sus TTPs (cómo actúa) le obliga a cambiar toda su metodología. Las mejores detecciones del SIEM se basan en comportamientos, no en IOCs estáticos.

> **GreyNoise** es una herramienta que muchos analistas desconocen pero que reduce enormemente los falsos positivos: diferencia entre los miles de escáneres automáticos de internet (Shodan, Censys, investigadores...) y los ataques reales dirigidos contra la organización.

> Los IOCs de un feed de Threat Intelligence **tienen fecha de caducidad**. Una IP maliciosa de hace 6 meses puede ser hoy un servidor legítimo que cambió de propietario. Siempre verificar con múltiples fuentes y considerar cuándo fue reportado el IOC antes de tomar acciones basadas en él.
