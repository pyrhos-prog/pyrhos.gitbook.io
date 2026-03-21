---
icon: shield-halved
---

# Tipos y Roles de SOC

### Roles dentro del SOC

#### Analista de SOC

El rol más común y el punto de entrada a la carrera en Blue Team. Se divide en tres niveles según la complejidad de los incidentes que gestiona:

| Nivel                      | Responsabilidades                                                                                                   | Certificaciones típicas |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| **L1 — Triaje**            | Monitorizar el SIEM, clasificar alertas (FP o amenaza real), recopilar información inicial, escalar a L2            | Security+, BTL1, eJPT   |
| **L2 — Investigación**     | Investigar alertas escaladas, análisis forense en endpoints con EDR, análisis de malware básico, determinar alcance | CySA+, GCIH, BTL2       |
| **L3 — Análisis avanzado** | Incidentes más complejos, threat hunting proactivo, ingeniería inversa de malware, crear reglas de detección        | GCFA, GREM              |

#### Respondedor de Incidentes (Incident Responder)

Gestiona el ciclo de vida completo de un incidente: evaluación inicial del alcance, coordinación de la contención, erradicación de la causa raíz, recuperación y análisis post-mortem. La diferencia clave con el analista es que **el analista detecta e investiga, el respondedor actúa**.

El proceso sigue el framework NIST SP 800-61: preparación → detección → contención → erradicación → recuperación → lecciones aprendidas.

#### Threat Hunter

Busca de forma **proactiva** amenazas que han evadido la detección automática. No espera alertas — las busca antes de que existan. Trabaja con hipótesis basadas en MITRE ATT\&CK y Threat Intelligence, y si encuentra algo lo escala a Incident Response. Si no encuentra nada, crea una regla de detección para automatizarlo.

La filosofía del threat hunter es _"Assume Breach"_: asumir que ya puede haber un atacante dentro y buscar evidencias de su actividad antes de que cause daño.

#### Ingeniero de Seguridad SOC

Mantiene y optimiza la infraestructura del SOC: configura el SIEM (reglas, correlaciones, dashboards), integra fuentes de logs, desarrolla playbooks de automatización en SOAR y reduce falsos positivos ajustando las reglas. La diferencia con el analista: **el analista usa las herramientas, el ingeniero las construye y mantiene**.

#### SOC Manager

Gestión del equipo (turnos, contratación, formación), presupuesto y planificación estratégica, comunicación con dirección en incidentes graves, definición de KPIs y coordinación con proveedores externos.

### Ruta de carrera en Blue Team

```
Entrada (0-2 años):
  Analista SOC L1

Intermedio (2-4 años):
  Analista SOC L2  ──── Ingeniero de Seguridad

Avanzado (4+ años):
  Analista L3 / Threat Hunter ──── Incident Responder Senior

Senior (6+ años):
  SOC Manager
```

### Blue Team vs Red Team vs Purple Team

| Equipo          | Rol                                           | Modo de trabajo                                   |
| --------------- | --------------------------------------------- | ------------------------------------------------- |
| **Blue Team**   | Defender la organización                      | Continuo, 24/7                                    |
| **Red Team**    | Simular ataques reales                        | Por proyectos y ejercicios periódicos             |
| **Purple Team** | Colaboración Red + Blue para mejorar defensas | El Red Team ataca y el Blue mejora en tiempo real |

> Tener conocimientos de Red Team siendo analista SOC es una ventaja enorme: entender cómo ataca un adversario permite buscar sus huellas con mucho más criterio que alguien que solo conoce el lado defensivo.

> La rotación en posiciones L1 es alta porque el trabajo puede volverse repetitivo. Los analistas que progresan son los que van más allá del triaje básico y aprenden el contexto técnico de cada alerta — no solo marcarla como FP o escalarla sin entenderla.
