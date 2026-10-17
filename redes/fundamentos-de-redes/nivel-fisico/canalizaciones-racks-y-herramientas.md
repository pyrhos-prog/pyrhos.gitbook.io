# Canalizaciones, racks y herramientas

### Canalizaciones

| Situación                          | Solución                                                         | Ventaja                           |
| ---------------------------------- | ---------------------------------------------------------------- | --------------------------------- |
| Edificio con canalización prevista | Falsos techos, suelos técnicos, conductos, cajas de registro     | Cableado oculto y accesible       |
| Sin canalización interior          | Canaletas de superficie, rodapiés técnicos, bandejas portacables | Protege, ordena y permite ampliar |

{% hint style="info" %}
Dejar siempre puntos de acceso (cajas de registro) y espacio libre para futuras ampliaciones. Cables sueltos sujetos a la pared o paredes cerradas sin registro son malas soluciones.
{% endhint %}

### Patch panel, rack y latiguillos

| Elemento                | Función                                                                   |
| ----------------------- | ------------------------------------------------------------------------- |
| Panel de parcheo        | Elemento pasivo donde termina el cableado fijo; no hace funciones lógicas |
| Rack                    | Armario estandarizado para paneles, switches, routers, servidores y SAI   |
| Unidad U                | Medida normalizada de altura en un rack (1U = 44,45 mm)                   |
| Latiguillo (patch cord) | Cable corto y flexible entre panel y switch                               |

Activar o mover una toma se hace cambiando el latiguillo, sin tocar el cableado fijo.

{% hint style="info" %}
Buenas prácticas de rack: patch panel arriba, switch debajo (o intercalado con guía-cables), peso abajo, etiquetado en panel y latiguillo, y SAI en la parte inferior.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (68).png" alt="" width="563"><figcaption></figcaption></figure>

### Herramientas

#### Par trenzado

| Herramienta        | Uso                                                    |
| ------------------ | ------------------------------------------------------ |
| Crimpadora         | Colocar conectores RJ45 en latiguillos a medida        |
| Impactadora        | Insertar y fijar conductores en paneles y rosetas      |
| Tester de cableado | Continuidad, orden de pares, hilos abiertos o cruzados |

#### Fibra óptica

| Herramienta                      | Uso                                                                      |
| -------------------------------- | ------------------------------------------------------------------------ |
| Cleaver (cortadora de precisión) | Corte limpio y perpendicular antes del empalme                           |
| Limpieza e inspección            | Revisar y limpiar los extremos (alcohol isopropílico cuando corresponde) |
| Fusionadora                      | Une dos fibras por arco eléctrico; menor pérdida que un empalme mecánico |
| Medidor de potencia óptica       | Mide pérdidas                                                            |
| OTDR                             | Localiza fallos y mide atenuación a lo largo del enlace                  |

{% hint style="danger" %}
la suciedad en el conector es la causa más frecuente de pérdidas. Inspeccionar y limpiar antes de culpar al enlace.
{% endhint %}
