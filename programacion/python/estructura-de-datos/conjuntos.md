# Conjuntos

Un conjunto guarda valores **únicos** y **sin orden**. Puedes añadir y quitar elementos, pero cada elemento debe ser inmutable.

Ejemplo: `conjunto = {1, 2, 3}`

### Operaciones con Conjuntos

#### Añadir y eliminar elementos

* Añadir: `conjunto.add(4)`
* Añadir varios: `conjunto.update({5,6})`
* Eliminar: `conjunto.remove(2)`
* Eliminar sin error: `conjunto.discard(10)`

#### Operaciones entre conjuntos

* Unión: `a.union(b)` → todos los elementos
* Intersección: `a.intersection(b)` → elementos comunes
* Diferencia: `a - b` → elementos de a que no están en b
* Diferencia simétrica: `a ^ b` → elementos que están en uno solo de los conjuntos

Ejemplo práctico:

`#!/usr/bin/env python3.13`

```python
mi_conjunto = {1, 2, 3}mi_segundo_conjunto = {3, 1, 7, 8}
mi_conjunto.add(4)mi_conjunto.update({5,6})
mi_conjunto.remove(2)mi_conjunto.discard(10)
interseccion_conjuntos = mi_conjunto.intersection(mi_segundo_conjunto)
union_conjuntos = mi_conjunto.union(mi_segundo_conjunto)
print(mi_conjunto.issubset(mi_segundo_conjunto))
print(f"mi conjunto intersección: {interseccion_conjuntos}")
print(f"mi conjunto unión: {union_conjuntos}")
```

#### Pruebas de pertenencia

Ejemplo: `3 in conjunto` devuelve `True` o `False`

#### Conjuntos inmutables

`frozenset` es un conjunto que no se puede modificar.

Ejemplo: `fs = frozenset([1,2,3])`

### Usos Comunes

* **Eliminar duplicados**: `set(lista)`
* **Comparar colecciones**: elementos comunes o diferentes
* **Búsquedas rápidas**: más eficiente que listas para muchos datos

### Buenas Prácticas

* Usar conjuntos para valores únicos y cuando no importa el orden.
* Ideales para comprobar rápidamente si un elemento está presente.
* Evitar conjuntos si se necesita mantener un orden fijo.
