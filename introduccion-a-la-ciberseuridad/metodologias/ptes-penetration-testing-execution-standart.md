---
icon: arrow-progress
---

# PTES (Penetration Testing Execution Standart)

> Metodología estándar para la ejecución de pruebas de penetración, dividida en 7 fases bien definidas.

### Fases de PTES

{% stepper %}
{% step %}
#### Interacciones previas al compromiso

* Contacto inicial con el cliente
* Definición del alcance (scope)
* Reglas de compromiso
* Contratos legales
* Acuerdo de No Divulgación (NDA)
{% endstep %}

{% step %}
#### Recopilación de información

* Reconocimiento pasivo
* Reconocimiento activo
* OSINT
* Enumeración inicial de activos
{% endstep %}

{% step %}
#### Threat Modeling

* Identificación de activos críticos
* Análisis de posibles vectores de ataque
* Priorización de objetivos
* Evaluación del impacto potencial
{% endstep %}

{% step %}
#### Análisis de vulnerabilidades

* Escaneo de vulnerabilidades
* Identificación de servicios vulnerables
* Validación de vulnerabilidades detectadas
{% endstep %}

{% step %}
#### Explotación

* Explotación controlada de vulnerabilidades
* Obtención de acceso inicial
* Verificación del impacto real
{% endstep %}

{% step %}
#### Post-explotación

* Escalada de privilegios
* Movimiento lateral
* Persistencia
* Extracción de información sensible
* Evaluación del impacto completo
{% endstep %}

{% step %}
#### Reporte

* Documentación de hallazgos
* Evidencias técnicas
* Riesgos identificados
* Recomendaciones y mitigaciones
{% endstep %}
{% endstepper %}
