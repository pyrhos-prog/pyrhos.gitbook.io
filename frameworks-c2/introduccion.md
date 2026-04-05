---
icon: transmission
---

# Introducción

## C2 Frameworks

Un framework de Command & Control (C2) es la infraestructura que permite a un operador de red team mantener acceso persistente a los sistemas comprometidos, ejecutar comandos, moverse lateralmente y exfiltrar datos durante un engagement. El C2 es el núcleo de cualquier operación ofensiva real: define qué capacidades tiene el operador una vez dentro, cómo se comunica el implante con el servidor y qué tan detectable es esa comunicación.

### Conceptos clave

**Implante / Agente** — el software que se ejecuta en el host comprometido. Puede ser una session (conexión interactiva continua) o un beacon (check-in periódico con sleep configurable).

**Listener** — el servicio en el servidor C2 que espera conexiones entrantes de los implantes. Define el protocolo de transporte: HTTP/S, DNS, mTLS, etc.

**Redirector** — servidor intermediario entre el implante y el C2 real. Dificulta la atribución y protege la infraestructura de operaciones si el tráfico es detectado.

**Sleep y Jitter** — el beacon duerme un tiempo fijo (sleep) más una variación aleatoria (jitter) entre check-ins. Reduce enormemente la detectabilidad por patrones de red.

### Frameworks cubiertos

| Framework                    | Licencia              | Lenguaje implante     | Detección           |
| ---------------------------- | --------------------- | --------------------- | ------------------- |
| **Sliver**                   | Open source (MIT)     | Go                    | Media               |
| **Cobalt Strike**            | Comercial (\~$5k/año) | C/C++                 | Alta (muy conocido) |
| **Havoc**                    | Open source           | C                     | Media-baja          |
| **Metasploit / Meterpreter** | Open source           | C/Ruby                | Alta                |
| **Mythic**                   | Open source           | Múltiple (por agente) | Variable            |
| **Brute Ratel C4**           | Comercial             | C                     | Baja-media          |

> En CTFs y labs, Metasploit y Sliver son las opciones más accesibles. En engagements reales, Cobalt Strike sigue siendo el estándar de la industria aunque su detección es alta por ser el más estudiado por los defensores. Havoc y Brute Ratel están ganando popularidad precisamente por su menor tasa de detección.
