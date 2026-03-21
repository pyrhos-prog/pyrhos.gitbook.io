# Errores Comunes de los Analistas de SOC

Los errores en un SOC pueden tener consecuencias graves:&#x20;

* un atacante permaneciendo semanas porque sus actividades se cerraron como falsos positivos.&#x20;
* datos robados que nadie detectó porque la alerta parecía de baja prioridad,&#x20;
* un incidente que escala porque no se documentó correctamente.&#x20;

Conocer los errores más frecuentes antes de cometerlos es una de las formas más eficientes de mejorar.

### Error 1 — Cerrar alertas como FP sin investigar adecuadamente

El analista ve "5 logins fallidos para usuario admin" y concluye "probablemente alguien olvidó la contraseña" sin investigar más. El problema: ¿y si el intento número 6 tuvo éxito? ¿Y si esa IP tiene score 95 en AbuseIPDB?

**Antes de cerrar cualquier alerta como FP:**

* ¿He buscado la IP/dominio/hash en al menos una fuente de TI?
* ¿He mirado si hay actividad relacionada en las últimas 24h?
* ¿Si esto fuera el inicio de un ataque real, qué más buscaría? → Buscarlo antes de cerrar.
* ¿Puedo justificar por escrito por qué es un FP? Si no puedo explicarlo, no es seguro cerrarlo.

### Error 2 — Investigar el síntoma sin buscar la causa raíz

Un endpoint se conecta a una IP de C2 conocida. El analista bloquea la IP en el firewall y cierra la alerta. El problema: el malware que generó esa conexión **sigue en el sistema**. La siguiente comunicación usará otra IP y el ciclo se repite. Se trató el síntoma (la conexión), no la enfermedad (el malware).

Bloquear la IP es correcto, pero solo como contención temporal mientras se investiga y erradica la causa raíz: ¿qué proceso hizo la conexión? ¿cómo llegó ese proceso al sistema? ¿hay otros sistemas afectados?

### Error 3 — No considerar el contexto temporal

El mismo evento tiene significados muy diferentes según el contexto:

**Alerta:** "Nuevo usuario administrador creado en el Controlador de Dominio"

* Sin contexto → escalada crítica inmediata
* Con contexto → el equipo de IT tenía planificada la creación de una cuenta temporal para un proveedor a las 2am del domingo → FP documentado

Siempre verificar: ¿había un cambio programado? ¿hay una ventana de mantenimiento activa? ¿el equipo de IT realizó alguna tarea programada? Consultar el calendario de cambios antes de escalar.

### Error 4 — Ignorar alertas de baja severidad

La mentalidad equivocada: _"Solo es un Low, no merece mi tiempo."_

Los atacantes APT son pacientes y generan una cadena de actividades que individualmente parecen inocuas:

* Día 1: escaneo de puertos → alerta **Low** → ignorada
* Día 3: login exitoso desde IP inusual → alerta **Medium** → cerrada como FP rápidamente
* Día 5: comandos `whoami`/`ipconfig` → alerta **Low** → ignorada por baja severidad
* Día 14: exfiltración de datos → alerta **High** → ahora es crítico, pero el atacante lleva 14 días dentro

Si se hubieran correlacionado las alertas Low del día 1 con el acceso del día 3 y el descubrimiento del día 5, el atacante habría sido detectado el día 5.

**Al ver una alerta Low:** buscar otras alertas relacionadas del mismo origen en las últimas 24-48 horas. Las alertas Low deben analizarse como conjunto.

### Error 5 — Documentación pobre o inexistente

Lo que se ve frecuentemente: _"Revisado. FP."_ o _"No es relevante."_

El problema: el turno siguiente no sabe qué se investigó ni por qué. Si la misma IP aparece al día siguiente, se repite toda la investigación. Es imposible detectar patrones en incidentes relacionados.

**Documentación correcta:**

> _"Alerta: bruteforce SSH desde 185.220.101.47 (20 intentos, ningún éxito). Consultado en AbuseIPDB: score 73/100, 847 reportes de SSH bruteforce. Revisados logs de auth.log: no hay login exitoso desde esa IP. Determinado: TRUE POSITIVE de bruteforce sin éxito. IP bloqueada en firewall. Añadida IP a watchlist 30 días. Notificado al equipo SIEM para evaluar si bloquear el rango del ASN."_

### Error 6 — No reportar FPs recurrentes al equipo SIEM

El analista identifica un FP, lo cierra correctamente... y al día siguiente otro analista abre la misma alerta del mismo FP. Nadie reportó el problema al equipo de ingeniería SIEM. La regla sigue generando el mismo FP semanas después.

Al cerrar un FP: buscar si ya hay un ticket abierto sobre ese FP. Si no lo hay → crear uno con la regla que lo genera, el contexto en que ocurre, la frecuencia, y una propuesta de ajuste.

### Error 7 — Escalar sin contexto

El analista L1 recibe una alerta compleja, no sabe qué hacer y escala al L2 con: _"Esta alerta parece rara, échale un vistazo."_

El L2 tiene que empezar la investigación desde cero. El tiempo se duplica. La información que el L1 tenía se pierde.

**Escalado correcto:** incluir toda la información recopilada hasta ese momento, explicar qué se investigó y qué se encontró (o no), indicar claramente por qué se escala y cuál es el nivel de urgencia estimado.

### Error 8 — Contener sin investigar

El analista encuentra un endpoint comprometido, lo aísla del EDR y da el caso por cerrado. El problema: la contención es solo el primer paso. Sin investigar: ¿cómo llegó el malware? ¿hay otros sistemas comprometidos? ¿el atacante tiene persistencia adicional?

El proceso correcto sigue el NIST SP 800-61: detección → contención → **investigación** → búsqueda lateral → erradicación → recuperación → post-mortem. Contener sin investigar es poner un parche en una tubería rota sin saber cuántas más están rotas.

### Error 9 — Ignorar las zonas horarias en los logs

Sistema A (servidor en UK): log con timestamp `14:00 UTC`. Sistema B (workstation en Madrid): log con timestamp `15:00 CEST (UTC+1)`. Son el mismo momento. Si el analista no tiene esto en cuenta, construye una línea de tiempo incorrecta y puede descartar una relación causal real entre eventos que en realidad son simultáneos.

Siempre verificar en qué zona horaria está cada fuente de logs y normalizar a UTC. Sospechar de diferencias de exactamente 1, 2, 3... horas entre eventos relacionados.

### Error 10 — No mantenerse actualizado

La ciberseguridad es el campo que más rápido evoluciona en IT. Un analista que solo sabe detectar las técnicas de hace 2 años tiene puntos ciegos para las técnicas actuales.

**30 minutos semanales de lectura del sector son suficientes:**

* [The Hacker News](https://thehackernews.com/) / [Bleeping Computer](https://bleepingcomputer.com/)
* [INCIBE-CERT avisos](https://incibe.es/incibe-cert)
* [CISA Known Exploited Vulnerabilities](https://cisa.gov/known-exploited-vulnerabilities-catalog)
* Informes gratuitos de Unit42 (Palo Alto) y Mandiant/Google TAG

### Resumen

| Error                       | Consecuencia                    | Solución                               |
| --------------------------- | ------------------------------- | -------------------------------------- |
| Cerrar FP sin investigar    | Ataque real no detectado        | 4 preguntas antes de cerrar            |
| Síntoma sin causa raíz      | Malware sigue activo            | Buscar siempre el proceso origen       |
| Ignorar contexto temporal   | FPs y FNs por falta de contexto | Consultar calendario de cambios        |
| Ignorar alertas Low         | Atacante APT no detectado       | Correlacionar Lows del mismo origen    |
| Documentación pobre         | Trabajo duplicado, sin contexto | Plantilla mínima para cada alerta      |
| No reportar FPs recurrentes | Fatiga de alertas del equipo    | Crear ticket al equipo SIEM            |
| Escalar sin contexto        | Tiempo duplicado                | Incluir todo lo investigado al escalar |
| Contener sin investigar     | Recaída, alcance desconocido    | Seguir el proceso NIST completo        |
| Ignorar zonas horarias      | Timelines incorrectos           | Normalizar a UTC, verificar siempre    |
| No actualizarse             | Puntos ciegos en la detección   | 30 min/semana de lectura               |

> El error más costoso no es el técnico sino el de juicio — cerrar una alerta como FP cuando es un TP real. Los errores técnicos se corrigen con experiencia, pero el error de juicio se corrige con metodología. Seguir siempre el proceso, aunque la alerta parezca obvia.

> Si alguna vez tienes dudas sobre si cerrar una alerta como FP, **la respuesta correcta es no cerrarla aún**. Un FP abierto extra no causa daño. Un TP cerrado como FP puede costarle a la organización semanas de tiempo de un atacante dentro de la red.
