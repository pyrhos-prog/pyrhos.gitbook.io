# Nessus — Evaluación de Habilidades y Referencia Rápida

### Conceptos clave para la evaluación

#### Sobre el escaneo

El escáner Nessus ejecuta plugins contra los hosts objetivo. Cada plugin comprueba una condición específica — una versión de software, una configuración, una respuesta de red — y genera un hallazgo si la condición es de riesgo.

Los resultados de un escaneo **sin credenciales** solo muestran lo que es visible desde la red. Los escaneos **con credenciales** tienen acceso al interior del sistema y detectan vulnerabilidades que no son visibles remotamente.

#### Sobre los plugins

Cada hallazgo en Nessus está generado por un **Plugin ID** único. El Plugin ID identifica exactamente qué comprobación se ejecutó. Desde el detalle del hallazgo se puede ver:

* El Plugin ID y su familia
* La descripción técnica de la vulnerabilidad
* La solución exacta recomendada
* Los CVE IDs asociados y el CVSS Score
* El output específico que generó la detección en ese host

#### Sobre CVSS

El **CVSS (Common Vulnerability Scoring System)** es el estándar para puntuar vulnerabilidades del 0 al 10. Nessus muestra tanto CVSS v2 como v3.

El vector CVSS describe las características de la vulnerabilidad:

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

AV (Attack Vector):    N=Network, A=Adjacent, L=Local, P=Physical
AC (Attack Complexity): L=Low, H=High
PR (Privileges Required): N=None, L=Low, H=High
UI (User Interaction): N=None, R=Required
S (Scope):            U=Unchanged, C=Changed
C/I/A (Impact):       N=None, L=Low, H=High
```

Un vector `AV:N/AC:L/PR:N/UI:N` indica explotación remota sin privilegios ni interacción del usuario — el escenario más crítico.

### Preguntas tipo de la evaluación

Las evaluaciones de HTB sobre Nessus suelen pedir:

* Número total de hosts escaneados y activos
* Número de hallazgos por severidad (Critical, High, Medium, Low, Info)
* Plugin ID de un hallazgo específico
* CVE asociado a una vulnerabilidad concreta
* Versión de software detectada como vulnerable
* Puerto en que se detectó un servicio específico
* Solución recomendada para un hallazgo

Para responder estas preguntas, usar la combinación de:

1. **Vista By Host** — ver todos los hallazgos de un host específico
2. **Vista By Plugin** — buscar un CVE o vulnerabilidad específica
3. **Filtros de severidad** — aislar Critical/High para contar
4. **Detalle del hallazgo** → pestaña Output — evidencia exacta con versión detectada

### Flujo de trabajo completo en un entorno de lab

```
1. Lanzar Nessus: systemctl start nessusd → https://localhost:8834

2. Crear escaneo:
   New Scan → Basic Network Scan
   → Name: "Lab Scan"
   → Targets: IP o rango del lab

3. Si hay credenciales disponibles:
   Pestaña Credentials → SSH o Windows → introducir datos
   (Los resultados serán mucho más completos)

4. Lanzar el escaneo → esperar a que termine

5. Analizar resultados:
   → Filtrar por Critical + High primero
   → Por cada hallazgo: leer Description + Output + Solution
   → Anotar Plugin ID, CVE y versión afectada

6. Exportar resultados:
   → .nessus para referencia
   → CSV para análisis
   → PDF para informe

7. Anotar falsos positivos detectados para crear Plugin Rules
```

### Referencia rápida

| Acción                           | Dónde                                |
| -------------------------------- | ------------------------------------ |
| Crear nuevo escaneo              | My Scans → New Scan                  |
| Ver resultados por host          | Clic en el escaneo → Hosts           |
| Ver resultados por plugin        | Clic en el escaneo → Vulnerabilities |
| Filtrar por severidad            | Barra de filtros → Severity          |
| Ver detalle de un hallazgo       | Clic sobre el hallazgo               |
| Ver evidencia de detección       | Detalle del hallazgo → Output        |
| Exportar resultados              | Escaneo → Export → formato deseado   |
| Crear política reutilizable      | Policies → New Policy                |
| Modificar severidad de un plugin | Settings → Plugin Rules              |
| Actualizar plugins manualmente   | Settings → Software Update           |
| Comparar dos escaneos            | Seleccionar ambos → Compare          |

### Diferencias entre Nessus y OpenVAS/GVM

|                   | Nessus                                       | OpenVAS / GVM                          |
| ----------------- | -------------------------------------------- | -------------------------------------- |
| **Licencia**      | Propietaria (Essentials gratis, Pro de pago) | Open source (100% gratuito)            |
| **Nº de plugins** | +180.000 (base de datos propia Tenable)      | +80.000 (Greenbone Feed)               |
| **Precisión**     | Alta, referencia del sector                  | Buena, mejora constante                |
| **Interfaz**      | Web moderna, intuitiva                       | Web funcional, menos pulida            |
| **Integración**   | Tenable.io, Tenable.sc                       | Greenbone Security Manager             |
| **Uso típico**    | Auditorías profesionales, pentesting         | Laboratorios, entornos sin presupuesto |
| **Instalación**   | Sencilla (paquete .deb/.rpm)                 | Más compleja (múltiples servicios)     |

Para el sector profesional, Nessus Professional es el estándar. Para aprendizaje y entornos de laboratorio, cualquiera de los dos es válido.

> En una auditoría real, los resultados de Nessus son el punto de partida, no el punto final. Cada hallazgo Critical debe verificarse manualmente antes de incluirlo en el informe — los falsos positivos existen y reportar uno sin verificar daña la credibilidad del analista.
