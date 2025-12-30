# Métodos especiales

Son métodos con doble guion bajo.\
Permiten que los objetos se comporten como tipos básicos.

| Método     | Para qué sirve                |
| ---------- | ----------------------------- |
| `__init__` | Crear el objeto               |
| `__str__`  | Mostrar el objeto con `print` |
| `__repr__` | Representación técnica        |
| `__eq__`   | Comparar con `==`             |
| `__lt__`   | Comparar con `<`              |
| `__add__`  | Usar `+`                      |

#### `__init__` y `__str__`

```python
class Persona:
    def __init__(self, nombre):
        self.nombre = nombre

    def __str__(self):
        return f"Persona: {self.nombre}"

p = Persona("Ana")
print(p)
```

`__str__` define qué se muestra cuando se imprime el objeto.

#### Comparación de objetos

```python
class Persona:
    def __init__(self, edad):
        self.edad = edad

    def __eq__(self, other):
        return self.edad == other.edad

    def __lt__(self, other):
        return self.edad < other.edad
```

Ahora los objetos se pueden comparar como números.

#### Operadores aritméticos

```python
class Numero:
    def __init__(self, valor):
        self.valor = valor

    def __add__(self, other):
        return Numero(self.valor + other.valor)

a = Numero(3)
b = Numero(5)
c = a + b
print(c.valor)
```
