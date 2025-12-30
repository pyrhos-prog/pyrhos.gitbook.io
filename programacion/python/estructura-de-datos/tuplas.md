---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/z0iyBve0fcLSJpEvb9YL/arquitecturas-de-red/arquitectura-p2p
---

# Tuplas

Las tuplas son colecciones ordenadas de elementos que no pueden modificarse una vez creadas, Se utilizan para que ciertos datos permanezcan constantes e inmutables.



### Características de las tuplas&#x20;

{% stepper %}
{% step %}
### Inmutabilidad

No se puede cambiar, añadir o eliminar los elementos de una tupla.
{% endstep %}

{% step %}
### Indexación y Slicing

Puedes acceder a los elementos mediante indices y tambien puedes realizar slicing para obtener subsecuencias de la tupla.
{% endstep %}

{% step %}
### Heterogeneidad

Pueden contener elementos de diferentes tipos, como otras tuplas.
{% endstep %}
{% endstepper %}

### Operaciones con Tuplas

#### Empaquetado y Desempaquetado

Guardar varios valores en una tupla y luego separarlos en variables.

Ejemplos:

* `datos = (10, 20)`
* `a, b = datos`
* `x, y, z = (1, 2, 3)`

#### Concatenación y Repetición

Unir o repetir tuplas.

Ejemplos:

* `(1, 2) + (3, 4)`
* `("a", "b") * 3`

#### Métodos de Búsqueda

| Método    | Uso básico                            | Ejemplo          |
| --------- | ------------------------------------- | ---------------- |
| `index()` | Devuelve la posición de un valor      | `tupla.index(2)` |
| `count()` | Cuenta cuántas veces aparece un valor | `tupla.count(1)` |

***

### Usos Comunes

* **Asignar varios valores**: `a, b = (5, 10)`
* **Devolver varios datos**: `return (x, y)`
* **Datos que no cambian**: `dias = ("lunes", "martes")`

***

### Buenas Prácticas

* Usar tuplas para datos fijos.
* Usarlas cuando no se necesiten cambios.
* Usar listas solo si los datos deben modificarse.
