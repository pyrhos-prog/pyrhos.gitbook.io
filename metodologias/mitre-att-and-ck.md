---
icon: arrow-progress
---

# MITRE ATT\&CK

## MITRE ATT\&CK

> MITRE ATT\&CK es un marco de conocimiento que documenta tácticas, técnicas y procedimientos (TTPs) utilizados por atacantes reales a lo largo del ciclo de un ataque.

No es una metodología de pentesting, sino una base de conocimiento utilizada para detección, defensa, simulación y análisis de amenazas.

### Objetivo de MITRE ATT\&CK

* Describir cómo operan los atacantes reales
* Estandarizar técnicas de ataque
* Mejorar la detección y respuesta
* Apoyar Blue Team, Red Team y SOC

### Estructura de MITRE ATT\&CK

MITRE ATT\&CK se organiza en **tácticas**, **técnicas** y **subtécnicas**.

#### Tácticas

Representan el **objetivo del atacante** en una fase concreta del ataque.

Ejemplos:

* Initial Access
* Privilege Escalation
* Lateral Movement
* Exfiltration

#### Técnicas

Describen **cómo** se logra ese objetivo.

Ejemplo:

* Phishing
* Exploitation for Privilege Escalation

#### Subtécnicas

Detallan variantes específicas de una técnica.

### Tácticas principales (Enterprise)

MITRE ATT\&CK define las siguientes tácticas en entornos empresariales:

* Reconnaissance
* Resource Development
* Initial Access
* Execution
* Persistence
* Privilege Escalation
* Defense Evasion
* Credential Access
* Discovery
* Lateral Movement
* Collection
* Command and Control
* Exfiltration
* Impact

### Matrices MITRE ATT\&CK

MITRE ATT\&CK se divide en varias matrices según el entorno:

* **Enterprise**: sistemas Windows, Linux, macOS, cloud
* **Mobile**: Android y iOS
* **ICS**: sistemas industriales

### Uso de MITRE ATT\&CK

MITRE ATT\&CK se utiliza para:

* Mapear ataques reales
* Evaluar detecciones
* Simular adversarios
* Mejorar reglas SIEM
* Threat Hunting
* Formación de equipos SOC

### MITRE ATT\&CK en Blue Team

* Detección basada en comportamiento
* Creación de reglas de correlación
* Análisis de brechas de visibilidad
* Mejora continua de defensas

### MITRE ATT\&CK en Red Team

* Simulación realista de adversarios
* Diseño de escenarios de ataque
* Evaluación de capacidades defensivas
* Medición de respuesta del SOC
