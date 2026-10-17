# Conectores, rosetas y esquemas T568

### Conectores por medio

| Medio        | Conector           | Notas                                        |
| ------------ | ------------------ | -------------------------------------------- |
| Coaxial      | BNC, conector en T | Redes antiguas 10BASE2                       |
| Par trenzado | RJ45               | Terminación según T568A o T568B              |
| Fibra        | ST (bayoneta)      | Menos frecuente hoy, multimodo               |
| Fibra        | SC (push-pull)     | Mono y multimodo                             |
| Fibra        | LC (compacto)      | Switches, módulos SFP, CPD; mono y multimodo |

LC y SC son los habituales por tamaño, facilidad de uso y compatibilidad.

### Orden de hilos en RJ45

Vista del conector con la lengüeta hacia abajo y los contactos hacia ti: pin 1 a la izquierda.

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
A y B solo intercambian los pares verde y naranja.
{% endhint %}

### Directo o cruzado

| Extremo 1 | Extremo 2 | Resultado | Uso tradicional                     |
| --------- | --------- | --------- | ----------------------------------- |
| T568B     | T568B     | Directo   | PC a switch, PC a router            |
| T568A     | T568A     | Directo   | Igual, si toda la instalación usa A |
| T568A     | T568B     | Cruzado   | PC a PC, switch a switch            |

Hoy casi todas las interfaces tienen Auto MDI-X, que detecta el tipo de conexión; un cruzado ya no suele hacer falta. Un cable cruzado en un equipo moderno normalmente sigue funcionando.

Consejo de instalación: elegir un esquema (lo habitual es T568B) y usarlo en toda la instalación, rosetas y paneles incluidos.

### Fabricar un latiguillo

{% embed url="https://youtu.be/XpeKWEXw7HE" %}

### Verificación

| Herramienta              | Qué detecta                                                            |
| ------------------------ | ---------------------------------------------------------------------- |
| Tester LAN               | Continuidad, orden de pares, hilos abiertos, cruzados o mal conectados |
| Certificador profesional | Cumplimiento de categoría: atenuación, diafonía, longitud              |

{% hint style="danger" %}
Una terminación incorrecta provoca pérdidas de señal, errores o menor rendimiento, aunque el tester básico dé continuidad. Los pares partidos son el caso típico: pasa el test de continuidad, pero hay diafonía.
{% endhint %}
