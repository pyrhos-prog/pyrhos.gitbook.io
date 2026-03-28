---
icon: shield-halved
---

# Feed de Inteligencia sobre Amenazas (Threat Intelligence)

**Threat Intelligence (TI)** es el conocimiento basado en evidencias sobre amenazas existentes o emergentes que ayuda a tomar decisiones de seguridad informadas. Va más allá de los datos brutos para proporcionar contexto, mecanismos e implicaciones sobre las amenazas.

La diferencia entre dato, información e inteligencia:

* **Dato:** `45.33.32.156` (una IP, sin contexto)
* **Información:** `45.33.32.156` tiene score 87/100 en AbuseIPDB
* **Inteligencia:** `45.33.32.156` es el C2 del grupo APT28 que actualmente ataca infraestructuras energéticas europeas usando phishing con documentos Word que explotan CVE-2023-XXXX

### Tipos de Threat Intelligence

| Tipo            | Para quién                               | Contenido                                                                                   |
| --------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Estratégica** | CISO, dirección                          | Tendencias por sector, grupos APT activos, riesgos geopolíticos, impacto financiero         |
| **Táctica**     | Equipo técnico, ingenieros SOC           | TTPs de grupos de amenaza, herramientas que usan, fases del ataque mapeadas a MITRE ATT\&CK |
| **Operacional** | Analistas SOC                            | Campañas activas en curso, IOCs de ataques actuales, advertencias de amenazas inminentes    |
| **Técnica**     | Analistas SOC, herramientas de seguridad | IOCs concretos: hashes, IPs, dominios, URLs, firmas YARA, reglas Sigma                      |

### IOCs — Indicadores de Compromiso

| Categoría              | Ejemplos                                                             |
| ---------------------- | -------------------------------------------------------------------- |
| **Basados en archivo** | Hashes MD5/SHA1/SHA256, nombre y ruta del archivo, tamaño            |
| **Basados en red**     | IPs de C2, dominios, URLs, User-Agent strings, certificados TLS      |
| **Basados en host**    | Claves de registro, nombres de servicios, mutex, comandos ejecutados |

#### La Pirámide del Dolor

La pirámide del dolor muestra cuánto le cuesta al atacante cambiar cada tipo de IOC cuando el defensor lo detecta y bloquea:

| Nivel       | IOC                   | Esfuerzo para el atacante             |
| ----------- | --------------------- | ------------------------------------- |
| ⬆️ Muy alto | TTPs (cómo actúa)     | Cambiar toda la metodología de ataque |
| Alto        | Herramientas          | Modificar o reemplazar el malware     |
| Medio       | Artifacts de red/host | Tiempo de reconfiguración             |
| Bajo        | Dominios              | Registrar un nuevo dominio            |
| Muy bajo    | IPs                   | Cambiar de servidor en minutos        |
| Trivial     | Hashes                | Recompilar el binario cambia el hash  |

> **Implicación práctica:** las mejores reglas de detección se basan en **TTPs** (cómo actúa el atacante), no solo en IOCs específicos como hashes e IPs que el atacante puede cambiar en minutos.

### Fuentes gratuitas de Threat Intelligence

| Herramienta        | Uso                                                                   |
| ------------------ | --------------------------------------------------------------------- |
| **VirusTotal**     | Análisis con 70+ motores AV de archivos, URLs, IPs y dominios         |
| **AbuseIPDB**      | Score de reputación de IPs basado en reportes de la comunidad         |
| **URLScan.io**     | Escaneo y captura de pantalla de URLs sospechosas                     |
| **Shodan**         | Información de IPs: puertos abiertos, servicios, banners              |
| **GreyNoise**      | Diferencia entre ruido de internet y amenazas reales                  |
| **AlienVault OTX** | Plataforma colaborativa con colecciones de IOCs (pulses)              |
| **MalwareBazaar**  | Repositorio de muestras de malware con búsqueda por hash              |
| **URLhaus**        | URLs de distribución de malware activo                                |
| **ThreatFox**      | IOCs de malware activo: IPs C2, dominios, hashes                      |
| **MITRE ATT\&CK**  | Framework de TTPs de grupos APT, la referencia fundamental del sector |

### Cómo usa el analista la TI en el triaje

Cuando el analista tiene una IP sospechosa en una alerta, el flujo habitual es:

1. **AbuseIPDB** → ¿tiene reportes? ¿de qué tipo? Score 0 → probablemente legítima. Score 87 → reportada múltiples veces por SSH bruteforce.
2. **VirusTotal** → ¿algún AV la detecta? ¿con qué dominios o hashes se asocia?
3. **Shodan** → ¿qué servicios expone? ¿parece un hosting legítimo o un VPS de atacante?
4. **GreyNoise** → ¿es ruido de internet generalizado o amenaza dirigida?
5. **MITRE ATT\&CK** → si se conoce el grupo o la técnica, ¿qué otras técnicas usan habitualmente y qué más debería buscar?

Con el SOAR este proceso se automatiza en segundos para cada alerta. Sin SOAR, manualmente toma 5-10 minutos.

### Integración en el SIEM

Los feeds de TI se integran directamente en el SIEM para detección automática: se importan listas de IPs y dominios maliciosos conocidos, y el SIEM alerta cuando cualquier sistema se comunica con esos IOCs. Los feeds se actualizan diariamente o en tiempo real mediante los estándares **STIX** (formato para describir amenazas e IOCs) y **TAXII** (protocolo para compartirlos entre herramientas).

### MITRE ATT\&CK

MITRE ATT\&CK es una base de conocimiento de las tácticas y técnicas que usan los atacantes reales, organizada como una matriz:

| Táctica                         | Descripción                                       |
| ------------------------------- | ------------------------------------------------- |
| `TA0001` Reconocimiento         | Recopilar información antes del ataque            |
| `TA0003` Acceso inicial         | Phishing, exploits, credenciales robadas          |
| `TA0005` Persistencia           | Tareas programadas, servicios, claves de registro |
| `TA0008` Acceso a credenciales  | Dump de LSASS, Kerberoasting                      |
| `TA0010` Movimiento lateral     | PsExec, WMI, RDP, Pass-the-Hash                   |
| `TA0012` C2 (Command & Control) | Beaconing, DNS tunneling, HTTPS                   |
| `TA0014` Impacto                | Ransomware, destrucción de datos                  |

Técnicas más detectadas en un SOC: `T1059` (PowerShell, cmd), `T1003` (credential dumping), `T1078` (valid accounts), `T1053` (scheduled tasks), `T1486` (ransomware).

> Los IOCs de un feed de TI **tienen fecha de caducidad**. Una IP maliciosa de hace 6 meses puede ser hoy un servidor legítimo que cambió de propietario. Siempre verificar con múltiples fuentes y considerar cuándo fue reportado el IOC antes de tomar acciones.
