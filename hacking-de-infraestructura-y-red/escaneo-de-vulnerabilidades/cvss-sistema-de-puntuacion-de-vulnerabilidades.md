# CVSS — Sistema de Puntuación de Vulnerabilidades

El **Common Vulnerability Scoring System (CVSS)** es el estándar de la industria para calcular la gravedad de vulnerabilidades de seguridad. Produce una puntuación numérica del 0 al 10 que permite a las organizaciones priorizar qué vulnerabilidades remediar primero. Herramientas como Nessus aplican automáticamente estas puntuaciones a cada hallazgo.

### Niveles de gravedad

| Puntuación | Nivel        | Color en Nessus |
| ---------- | ------------ | --------------- |
| 9.0 – 10.0 | **Critical** | Rojo oscuro     |
| 7.0 – 8.9  | **High**     | Rojo            |
| 4.0 – 6.9  | **Medium**   | Naranja         |
| 0.1 – 3.9  | **Low**      | Amarillo        |
| 0.0        | **Info**     | Azul            |

### Los tres grupos de métricas

La puntuación CVSS se compone de tres grupos. El más importante es el grupo base — los otros dos ajustan la puntuación según el contexto temporal y el entorno específico de la organización.

#### 1. Grupo de métricas base

Representa las características intrínsecas de la vulnerabilidad, independientemente del entorno. Se divide en métricas de explotabilidad y métricas de impacto.

**Métricas de explotabilidad**

Miden los medios técnicos necesarios para explotar la vulnerabilidad:

| Métrica                      | Valores                               | Descripción                                                  |
| ---------------------------- | ------------------------------------- | ------------------------------------------------------------ |
| **Attack Vector (AV)**       | Network / Adjacent / Local / Physical | Dónde debe estar el atacante para explotar la vulnerabilidad |
| **Attack Complexity (AC)**   | Low / High                            | Condiciones adicionales necesarias para el éxito del ataque  |
| **Privileges Required (PR)** | None / Low / High                     | Nivel de acceso previo que necesita el atacante              |
| **User Interaction (UI)**    | None / Required                       | ¿Requiere que un usuario haga algo?                          |

**Métricas de impacto — Tríada CIA**

Miden el impacto sobre los tres pilares de la seguridad si la vulnerabilidad es explotada con éxito:

| Métrica                        | Valores           | Descripción                         |
| ------------------------------ | ----------------- | ----------------------------------- |
| **Confidentiality Impact (C)** | None / Low / High | Exposición de información protegida |
| **Integrity Impact (I)**       | None / Low / High | Modificación de datos o archivos    |
| **Availability Impact (A)**    | None / Low / High | Interrupción del acceso a servicios |

Para cada métrica de impacto, un valor **High** indica impacto total (pérdida completa de confidencialidad, integridad o disponibilidad) y **Low** indica impacto parcial sin control total por parte del atacante.

**Scope (S)**

Indica si la vulnerabilidad puede afectar a componentes más allá del sistema vulnerable directamente:

* **Unchanged** — el impacto se limita al componente vulnerable
* **Changed** — la explotación puede impactar otros componentes del entorno

#### 2. Grupo de métricas temporales

Detalla la disponibilidad de exploits y parches en el momento del análisis. Ajusta la puntuación base hacia abajo si la vulnerabilidad no tiene exploit disponible o ya tiene parche oficial.

**Madurez del código de explotación**

| Valor                | Significado                                                     |
| -------------------- | --------------------------------------------------------------- |
| **High**             | Exploit funcional y automatizable, ampliamente disponible       |
| **Functional**       | Código de explotación público y funcional                       |
| **Proof-of-Concept** | PoC disponible pero requiere adaptación para explotar con éxito |
| **Unproven**         | No hay exploit conocido o disponible públicamente               |
| **Not Defined**      | Se omite esta métrica                                           |

**Nivel de remediación**

| Valor             | Significado                                                             |
| ----------------- | ----------------------------------------------------------------------- |
| **Official Fix**  | El proveedor ha publicado un parche oficial                             |
| **Temporary Fix** | El proveedor ofrece una solución temporal mientras desarrolla el parche |
| **Workaround**    | Existe una mitigación no oficial                                        |
| **Unavailable**   | No hay ninguna solución disponible                                      |
| **Not Defined**   | Se omite esta métrica                                                   |

**Confianza en el reporte**

| Valor           | Significado                                                           |
| --------------- | --------------------------------------------------------------------- |
| **Confirmed**   | Múltiples fuentes con detalles técnicos confirman la vulnerabilidad   |
| **Reasonable**  | Información publicada pero sin confirmación total de reproducibilidad |
| **Unknown**     | Información no confirmada                                             |
| **Not Defined** | Se omite esta métrica                                                 |

#### 3. Grupo de métricas ambientales

Permite que cada organización ajuste la puntuación según la importancia de sus activos y su entorno específico. Utiliza las **Modified Base Metrics** para sobrescribir las métricas base con valores más apropiados para el contexto concreto.

| Nivel           | Significado                                                                                 |
| --------------- | ------------------------------------------------------------------------------------------- |
| **High**        | El impacto sobre la tríada CIA tendría efectos críticos para la organización y sus clientes |
| **Medium**      | El impacto sería significativo pero no catastrófico                                         |
| **Low**         | El impacto sería mínimo para la operación de la organización                                |
| **Not Defined** | Se omite esta métrica                                                                       |

### Leer el vector CVSS

El vector CVSS resume todas las métricas en una cadena de texto. Ejemplo real de PrintNightmare (CVE-2021-34527):

```
CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H

Desglose:
AV:N  → Attack Vector: Network        (explotable remotamente)
AC:L  → Attack Complexity: Low        (sin condiciones especiales)
PR:L  → Privileges Required: Low      (requiere usuario autenticado)
UI:N  → User Interaction: None        (no requiere acción del usuario)
S:U   → Scope: Unchanged              (impacto limitado al componente)
C:H   → Confidentiality Impact: High  (pérdida total de confidencialidad)
I:H   → Integrity Impact: High        (pérdida total de integridad)
A:H   → Availability Impact: High     (pérdida total de disponibilidad)

Resultado: CVSS Base Score 8.8 — High
```

### DREAD — Sistema complementario de Microsoft

Junto con CVSS, Microsoft desarrolló **DREAD** como sistema complementario de evaluación de riesgos. Usa una escala de 10 puntos para cada uno de sus cinco factores:

| Factor               | Pregunta que responde                                           |
| -------------------- | --------------------------------------------------------------- |
| **D**amage Potential | ¿Cuánto daño puede causar si se explota?                        |
| **R**eproducibility  | ¿Con qué facilidad se puede reproducir el ataque?               |
| **E**xploitability   | ¿Qué conocimientos o herramientas se necesitan para explotarlo? |
| **A**ffected Users   | ¿Cuántos usuarios o sistemas se ven afectados?                  |
| **D**iscoverability  | ¿Qué tan fácil es descubrir la vulnerabilidad?                  |

La puntuación DREAD se calcula promediando los cinco valores. Se usa principalmente en el contexto de productos Microsoft y como complemento al análisis CVSS en evaluaciones de riesgo internas.

### Calculadora CVSS

La Base de Datos Nacional de Vulnerabilidades (NVD) del NIST ofrece una calculadora pública para CVSS v3.1:

`https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator`

> Nessus muestra tanto el CVSS v2 como el v3.1 para cada hallazgo. El v3 es más preciso y es el estándar actual — usar siempre v3 como referencia para priorizar.

> La puntuación CVSS refleja la severidad técnica de la vulnerabilidad en términos generales, no el riesgo real en un entorno específico. Una vulnerabilidad con CVSS 9.8 en un servidor aislado sin acceso a internet puede ser menos urgente que una con CVSS 6.5 en un sistema expuesto con datos sensibles. Las métricas ambientales y el contexto del negocio son esenciales para una priorización correcta.
