---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/z0iyBve0fcLSJpEvb9YL/arquitecturas-de-red/arquitectura-cliente-servidor
---

# Listas

Las listas son estructuras de datos ordenadas y mutables en Python que permiten almacenar múltiples elementos en una sola variable. Son ampliamente utilizadas por su flexibilidad y facilidad de uso.

### Características de las Listas

* **Datos heterogéneos**: pueden contener elementos de distintos tipos de datos.

```python
lista = [1, "texto", 3.14, True]
```

* **Indexación y corte**: permiten acceder a elementos específicos o a subconjuntos de la lista.

```
numeros = [10, 20, 30, 40]
numeros[1]      # 20
numeros[1:3]    # [20, 30]
```

* **Listas anidadas**: una lista puede contener otras listas.

```
matriz = [[1, 2], [3, 4]]
```



### Métodos de Listas

| Método      | Descripción general                           |
| ----------- | --------------------------------------------- |
| `append()`  | Añade un elemento al final de la lista.       |
| `extend()`  | Añade varios elementos a la lista.            |
| `insert()`  | Inserta un elemento en una posición indicada. |
| `remove()`  | Elimina un elemento por su valor.             |
| `pop()`     | Elimina un elemento según su posición.        |
| `clear()`   | Elimina todos los elementos de la lista.      |
| `index()`   | Devuelve la posición de un elemento.          |
| `count()`   | Cuenta cuántas veces aparece un elemento.     |
| `sort()`    | Ordena los elementos de la lista.             |
| `reverse()` | Invierte el orden de los elementos.           |

### Buenas Prácticas

* Usar listas cuando se necesiten colecciones ordenadas y modificables.
* Usar tuplas para datos inmutables.
* Usar conjuntos para evitar duplicados.
* Usar diccionarios para relaciones clave-valor.
