# Diccionarios



Un diccionario es una colección de pares **clave-valor**. Cada clave es única y se utiliza para acceder a su valor correspondiente. Los diccionarios son **desordenados** y **dinámicos**, lo que significa que puedes agregar, modificar y eliminar elementos.

Ejemplo básico:

```python
mi_diccionario = {"nombre": "Pyrhos", "edad": 19, "ciudad": "Madrid"}
```

### Operaciones con Diccionarios

#### Agregar y modificar elementos

```python
mi_diccionario["profesion"] = "Lammer"  # Agregar
mi_diccionario["edad"] = 20              # Modificar
```

#### Eliminar elementos

```python
del mi_diccionario["profesion"]          # Eliminar usando del
valor_eliminado = mi_diccionario.pop("edad")  # Eliminar y obtener el valor
```

## Diccionarios en Python

### Qué es un diccionario

Un diccionario almacena datos como pares **clave-valor**, donde cada clave identifica un valor. No tiene un orden fijo y se puede cambiar agregando, modificando o eliminando elementos.

Ejemplo:

```python
mi_diccionario = {"nombre": "Pyrhos", "edad": 19, "ciudad": "Madrid"}
```

Aquí "nombre", "edad" y "ciudad" son las claves, y cada una tiene un valor asociado.

***

### Operaciones básicas

#### Agregar o cambiar valores

```python
mi_diccionario["profesion"] = "Lammer"   # Se agrega un nuevo par clave-valor
mi_diccionario["edad"] = 20               # Se cambia el valor de la clave existente
```

#### Eliminar elementos

```python
del mi_diccionario["profesion"]           # Elimina el par clave-valor
mi_diccionario.pop("edad")                # También elimina y devuelve el valor eliminado
```

#### Acceder a valores

```python
print(mi_diccionario["nombre"])           # Se obtiene el valor de la clave indicada
```

#### Comprobar si existe una clave

```python
if "ciudad" in mi_diccionario:
    print("La clave existe")
```

#### Recorrer todo el diccionario

```python
for clave, valor in mi_diccionario.items():
    print(clave, valor)                       # Muestra todas las claves con sus valores
```

#### Saber cuántos elementos hay

```python
print(len(mi_diccionario))                   # Devuelve el número de pares clave-valor
```

#### Otros métodos útiles

```python
claves = mi_diccionario.keys()              # Obtiene todas las claves
valores = mi_diccionario.values()          # Obtiene todos los valores
pares = mi_diccionario.items()             # Devuelve pares clave-valor

# Se puede usar comprensión para crear un nuevo diccionario, por ejemplo multiplicando valores numéricos
nuevo_dicc = {k: v*2 for k, v in mi_diccionario.items() if isinstance(v, int)}
```

***

### Usos habituales

* Guardar datos que estén relacionados entre sí.
* Poder acceder a ellos rápidamente usando la clave.
* Se pueden tener valores complejos, como listas o diccionarios dentro de diccionarios.

***

### Recomendaciones

* Usar claves claras y únicas.
* Comprobar que una clave existe antes de acceder a ella para evitar errores.
* Aprovechar comprensiones de diccionarios para crear datos de forma rápida y sencilla.
