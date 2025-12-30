# Decoradores y propiedades

### Decoradores

Los decoradores permiten modificar el comportamiento de funciones o métodos sin cambiar su código directamente. Se usan para añadir lógica extra de forma ordenada.

Un decorador es una función que recibe otra función y devuelve una nueva.

Ejemplo básico:

```python
def mi_decorador(funcion):
    def envoltura():
        print("Antes")
        funcion()
        print("Después")
    return envoltura

@mi_decorador
def saludar():
    print("Hola")

saludar()
```

Aquí el decorador añade código antes y después de la función original.

***

### Propiedades

Las propiedades permiten acceder a métodos como si fueran atributos normales. Se usan sobre todo para controlar cómo se leen o modifican datos internos.

Se crean usando el decorador `@property`.

Ejemplo:

```python
class Usuario:
    def __init__(self, edad):
        self.__edad = edad

    @property
    def edad(self):
        return self.__edad
```

Ahora `edad` se puede usar como si fuera un atributo público.

***

### Getters y Setters

Un **getter** devuelve el valor de un atributo.\
Un **setter** permite modificarlo aplicando reglas o validaciones.

Ejemplo completo:

```python
class Usuario:
    def __init__(self, edad):
        self.__edad = edad

    @property
    def edad(self):
        return self.__edad

    @edad.setter
    def edad(self, valor):
        if valor >= 0:
            self.__edad = valor
```

Uso:

```python
u = Usuario(20)
print(u.edad)
u.edad = 25
```

***

### Deleter

El deleter define qué ocurre cuando se elimina un atributo.

```python
class Usuario:
    def __init__(self, edad):
        self.__edad = edad

    @property
    def edad(self):
        return self.__edad

    @edad.deleter
    def edad(self):
        print("Edad eliminada")
        del self.__edad
```

Uso:

```python
del u.edad
```
