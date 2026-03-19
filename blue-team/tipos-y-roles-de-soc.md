---
icon: shield-halved
---

# Tipos y Roles de SOC

### Tipos de modelos SOC

Dependiendo del tamaño, presupuesto y necesidades de seguridad de la organización, el SOC puede estructurarse de distintas formas.

#### SOC Interno

```
La organización construye y gestiona su propio equipo de seguridad.

Características:
→ Equipo dedicado exclusivamente a esa organización
→ Conocimiento profundo de la infraestructura interna
→ Respuesta más rápida (están dentro de la organización)
→ Requiere presupuesto elevado y sostenido
→ Difícil de justificar para organizaciones pequeñas

Adecuado para:
→ Grandes corporaciones y multinacionales
→ Sector financiero, defensa, infraestructuras críticas
→ Organizaciones con datos muy sensibles
Ejemplos: bancos grandes, operadoras de telecomunicaciones,
          agencias gubernamentales
```

#### SOC Virtual

```
No tiene instalación permanente. El equipo trabaja de forma remota
desde distintas ubicaciones.

Características:
→ Más flexible y económico que el SOC interno
→ Puede distribuirse geográficamente
→ Requiere buenas herramientas de colaboración y acceso remoto seguro
→ Comunicación más compleja que en un SOC centralizado

Adecuado para:
→ Organizaciones con equipos distribuidos
→ Empresas que han adoptado el modelo remoto/híbrido
→ Startups o empresas medianas con talento disperso
```

#### SOC Cogestionado (Managed SOC / MSSP)

```
Personal interno del SOC + MSSP (Managed Security Service Provider) externo.
La organización mantiene el control pero externaliza parte de las operaciones.

Características:
→ Combina conocimiento interno con experiencia externa especializada
→ El MSSP aporta: herramientas, personal 24/7, expertise sectorial
→ La coordinación entre equipos internos y externos es crítica
→ Más económico que un SOC interno completo

Adecuado para:
→ Medianas empresas que necesitan cobertura 24/7 sin ese coste
→ Organizaciones que quieren reforzar su equipo interno

Proveedores MSSP comunes en España:
→ S21sec, Tarlogic, Accenture Security, IBM MSS, Sophos MDR
```

#### SOC de Comando

```
Supervisa múltiples SOC más pequeños dentro de una región o
gran organización. Actúa como coordinador central.

Características:
→ Visión global de toda la postura de seguridad del grupo
→ Coordina respuestas a incidentes de gran escala
→ Comparte inteligencia de amenazas entre los SOC subordinados
→ Define las políticas y estándares para todos los SOC

Adecuado para:
→ Grandes multinacionales con filiales en múltiples países
→ Grandes proveedores de telecomunicaciones
→ Agencias de defensa nacional
```

### Roles dentro del SOC

#### Analista de SOC (L1 / L2 / L3)

El rol más común y el punto de entrada a la carrera en Blue Team.

```
Analista L1 — Triaje:
→ Primera línea. Recibe y clasifica las alertas del SIEM.
→ Determina si es un falso positivo o requiere investigación
→ Recopila información inicial: IP, usuario, hostname, hash...
→ Escala a L2 cuando el análisis supera su capacidad
→ Documenta cada alerta revisada

Analista L2 — Investigación:
→ Investiga los incidentes escalados por L1
→ Análisis forense inicial de endpoints con el EDR
→ Correlación de eventos a lo largo del tiempo
→ Análisis de malware básico: comportamiento, IOCs
→ Determina alcance e impacto del incidente

Analista L3 — Análisis avanzado:
→ Investiga los incidentes más complejos
→ Threat hunting proactivo (buscar amenazas sin alerta previa)
→ Ingeniería inversa de malware avanzado
→ Desarrolla nuevas reglas de detección para el SIEM
→ Mentoriza a L1 y L2
```

#### Respondedor de Incidentes (Incident Responder)

```
Responsabilidades:
→ Gestiona el ciclo de vida completo de un incidente de seguridad
→ Evaluación inicial del alcance e impacto de la brecha
→ Coordina la contención: aislar sistemas, bloquear IOCs
→ Erradicación de la causa raíz
→ Recuperación y restauración de sistemas
→ Análisis post-mortem

Diferencia con el analista:
→ El analista detecta e investiga
→ El respondedor actúa: contiene, erradica, recupera
```

#### Threat Hunter

```
Responsabilidades:
→ Busca de forma PROACTIVA amenazas que han evadido la detección
→ No espera alertas — las busca antes de que existan
→ Trabaja con hipótesis basadas en MITRE ATT&CK y Threat Intel
→ Si encuentra algo → escala a Incident Response
→ Si no encuentra nada → crea regla de detección para automatizarlo

Filosofía: "Assume Breach"
→ Asumir que ya puede haber un atacante dentro
→ Buscar evidencias de su actividad antes de que haga daño
```

#### Ingeniero de Seguridad SOC

```
Responsabilidades:
→ Mantiene y optimiza la infraestructura del SOC
→ Configura el SIEM: reglas, correlaciones, dashboards
→ Integra fuentes de logs con el SIEM
→ Desarrolla playbooks de automatización en SOAR
→ Reduce falsos positivos ajustando las reglas

Diferencia con el analista:
→ El analista usa las herramientas
→ El ingeniero las construye, configura y mantiene
```

#### SOC Manager

```
Responsabilidades:
→ Gestión del equipo: turnos, contratación, formación
→ Presupuesto y planificación estratégica del SOC
→ Comunicación con dirección en incidentes graves
→ Definición de KPIs y métricas del SOC
→ Coordinación con equipos externos (proveedores, CERT, Legal)
```

### Ruta de carrera en Blue Team

```
ENTRADA (0-2 años):
└── Analista SOC L1
    Certs recomendadas: CompTIA Security+, BTL1, eJPT (base ofensiva)
    Plataformas: LetsDefend, TryHackMe (SOC path), Blue Team Labs

INTERMEDIO (2-4 años):
├── Analista SOC L2
│   Certs: CySA+, GCIH, BTL2
│
└── Ingeniero de Seguridad
    Certs: Splunk Core, Microsoft Sentinel SC-200

AVANZADO (4+ años):
├── Analista L3 / Threat Hunter
│   Certs: GCFA, GREM, THP
│
├── Incident Responder Senior
│   Certs: GCFE, GCFA, CIRT
│
└── SOC Manager
    Certs: CISSP, CISM
```

### Diferencia entre SOC y Red Team

```
Blue Team (SOC):
→ Defiende la organización
→ Monitoriza, detecta y responde
→ Trabaja 24/7 de forma continua
→ Reactivo a amenazas + proactivo en hunting

Red Team:
→ Simula ataques reales contra la organización
→ Busca vulnerabilidades antes que los atacantes reales
→ Trabaja por proyectos o ejercicios periódicos
→ Completamente ofensivo

Purple Team:
→ Colaboración entre Red y Blue
→ El Red Team ataca y el Blue Team mejora sus defensas en tiempo real
→ Maximiza el aprendizaje de ambos equipos
→ Modelo más moderno y efectivo para mejorar la seguridad
```

> El analista L1 es el punto de entrada más accesible al sector de la ciberseguridad. Aunque puede parecer mecánico, es donde se desarrolla el ojo clínico para detectar comportamientos anómalos, que es la habilidad más valiosa en Blue Team.

> Tener conocimientos de Red Team (ofensivo) siendo analista SOC es una ventaja enorme: entender cómo ataca un adversario permite buscar sus huellas con mucho más criterio que alguien que solo conoce el lado defensivo.

> La rotación en posiciones L1 es alta porque el trabajo puede volverse repetitivo. Los analistas que progresan son los que van más allá del triaje básico y aprenden el contexto técnico de cada alerta — no solo marcarla como FP o escalarla sin entenderla.
