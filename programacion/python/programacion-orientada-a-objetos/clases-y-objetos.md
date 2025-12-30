---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/z0iyBve0fcLSJpEvb9YL/protocolos-de-transmision/protocolo-tcp
---

# Clases y objetos

Una clase es una forma de definir algo "en general". Sirve como plantilla para crear objetos. Dentro de una clase se indican los datos que tendrá (atributos) y lo que podrá hacer (métodos).

#### Definición

En Python, una clase se crea usando la palabra `class`.

```python
class Persona:
    pass
```

Esta clase todavía no hace nada, pero ya se puede usar para crear objetos.

#### Ejemplo sencillo

```python
class Persona:
    nombre = ""
```

Aquí la clase define un dato llamado `nombre`.

#### Constructor

El constructor es el método `__init__`. Se ejecuta automáticamente cuando se crea un objeto y sirve para darle valores iniciales.

```python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad
```

* `self` hace referencia al propio objeto
* `nombre` y `edad` quedan guardados en el objeto

### Objetos

#### Definición

Un objeto es una instancia de una clase. Es decir, una copia real creada a partir de la plantilla.

#### Ejemplo

```python
class Persona:
    def __init__(self, nombre):
        self.nombre = nombre

persona1 = Persona("Ana")
persona2 = Persona("Luis")
```

Aunque vienen de la misma clase, son objetos distintos.

#### Propiedades

Las propiedades son los datos guardados dentro del objeto.

```python
print(persona1.nombre)
```

Cada objeto tiene sus propios valores.

***

### Decoradores

#### Definición

Un decorador sirve para añadir comportamiento extra a una función o método sin modificar su código original.

#### Ejemplo básico

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

#### Decoradores comunes

* `@staticmethod`: define un método que no usa datos del objeto ni de la clase
* `@classmethod`: define un método que trabaja con la clase
* `@property`: permite acceder a un método como si fuera un atributo

Ejemplo de `@property`:

```python
class Persona:
    def __init__(self, edad):
        self._edad = edad

    @property
    def edad(self):
        return self._edad

persona = Persona(20)
print(persona.edad)
```
