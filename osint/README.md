---
icon: hand-fist
---

# Introducción al OSINT

## ¿Que es el OSINT?

El OSINT (Open Source Intelligence) es el uso de fuentes públicas y abiertas, para recopilar, analizar y explotar la información obtenida.

### Fuentes de información abierta

Cualquier cosa que sea de acceso publico es una fuente para el OSINT. las mas comunes son:

* Motores de búsqueda (Google Dorks), Redes sociales (Linkedin, Facebook, Twitter...), foros, blogs y la Deep web.
* Registros públicos como registros de DNS, Registros de negovios y de propiedad.
* Direcciones IP, puertos abiertos, metadatos de archivos, datos de geolocalización.
* Periódicos, revistas, radio y televisión.



### Fases del OSINT

| Fases                        | Descripción                                                                                                                                           | Preguntas Clave                                               |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| 1. Planificación / Dirección | Definir el objetivo de la investigación, sirve para establecer el alcance y el tipo de información necesaria.                                         | ¿Qué necesito saber?                                          |
| 2. Recolección / Obtención   | Recopilar la información del objetivo utilizando Google Dorks, Shodan, PimEyes, etc..                                                                 | ¿Dónde puedo encontrar esta información?                      |
| 3. Procesamiento / Filtrado  | Eliminar duplicados, información irrelevante o contradictoria para mejorar la estructura de los datos.                                                | ¿Este dato es relevante para la investigación?                |
| 4. Análisis y Correlación    | Interpretar los datos limpios, buscar patrones, conexiones, cronologías y contrastar la información para comprobar si es fiable y sacar conclusiones. | ¿Qué significa esta conexión? ¿Hay un patrón en la actividad? |
| 5. Difusión / Actuación      | Organizar la información encontrada de manera clara.                                                                                                  | ¿Es entendible para un público general?                       |

### Técnicas de Recolección

En la fase de recolección hay 3 niveles según el nivel de interacción:

* **Búsqueda Pasiva:** Se obtiene información sin que el objetivo se dé cuenta de que esta siendo investigado.
* **Búsqueda semi pasiva:** Implica una interacción mínima, como hacer una consulta a un servidor DNS. Puede ser visible en logs pero es dificil de rastrear al investigador.
* **Búsqueda Activa:** Interacción directa con el objetivo (visitar la web, intentar iniciar sesión con credenciales por defecto).
