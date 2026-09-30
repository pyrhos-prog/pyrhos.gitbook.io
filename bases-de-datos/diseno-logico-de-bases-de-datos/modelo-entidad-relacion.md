---
icon: database
---

# Modelo entidad-relación

> Permite representar de forma grafica y conceptual los elementos antes de construir una base de datos.

**Para ello se utiliza:**

* **Entidades:** Representan los objetos de interes&#x20;
* **Atributos:** Describen las caracteristicas de las entidades
* **Relaciones:** Muestra como se asocian entre si

#### Pasos para realizar un modelo entidad relación

1. Se parte de una descripción textual del problema o sistema de información a automatizar (los requisitos).
2. Se hace una lista de los sustantivos y verbos que aparecen.
3. Los sustantivos son posibles entidades o atributos.
4. Los verbos son posibles relaciones.
5. Analizando las frases se determina la cardinalidad de las relaciones y otros detalles.
6. Se elabora el diagrama (o diagramas) entidad-relación.
7. Se completa el modelo con listas de atributos y una descripción de otras restricciones que no se pueden reflejar en el diagrama.



### Entidades

> Representa un objeto, concepto o elemento relevante del que queremos almacenar información.
>
> Debe de ser distinguible mediante algun método de identificación

Según su capacidad para identificarse por sí misma, podemos diferenciar entre **entidades fuertes** y **entidades débiles**.

#### Entidades fuertes

Son entidades que se pueden identifcar por si mismas mediante uno o varios de sus propios atributos, sin necesidad de clave o de otra entidad.

Por ejemplo: en una base de datos de un centro educativo, una ocurrencia de la entidad **ALUMNO** puede identificarse mediante su propio identificador, independientemente de la entidad **CURSO**.                         &#x20;

#### Entidades débiles





