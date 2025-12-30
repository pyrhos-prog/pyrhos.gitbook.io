# Herencia y poliformismo

### Herencia

#### Qué es la herencia

La herencia permite crear una clase nueva a partir de otra ya existente. La nueva clase hereda atributos y métodos, y puede añadir o modificar comportamiento.

Se usa para reutilizar código y organizar clases relacionadas.

#### Ejemplo básico de herencia

```python
class Animal:
    def __init__(self, nombre):
        self.nombre = nombre

    def hablar(self):
        print("El animal hace un sonido")

class Perro(Animal):
    def hablar(self):
        print("El perro ladra")
```

Aquí:

* `Animal` es la clase base.
* `Perro` hereda de `Animal`.
* `Perro` redefine el método `hablar`.

***

#### Uso de métodos heredados

```python
perro = Perro("Rex")
print(perro.nombre)
perro.hablar()
```

El objeto usa atributos heredados y métodos propios.

***

#### Llamar a la clase padre con `super()`

```python
class Empleado:
    def __init__(self, nombre):
        self.nombre = nombre

class Programador(Empleado):
    def __init__(self, nombre, lenguaje):
        super().__init__(nombre)
        self.lenguaje = lenguaje
```

`super()` permite reutilizar el constructor de la clase padre.

### Polimorfismo

#### Qué es el polimorfismo

El polimorfismo permite usar el mismo método en clases diferentes, aunque cada una lo implemente a su manera.

#### Ejemplo de polimorfismo

```python
class Gato:
    def hablar(self):
        print("El gato maúlla")

class Perro:
    def hablar(self):
        print("El perro ladra")
```

Ambas clases tienen el método `hablar`.

#### Uso común del polimorfismo

```python
animales = [Perro(), Gato()]

for animal in animales:
    animal.hablar()
```

No importa la clase del objeto, mientras tenga el método esperado.

### Para qué se usan

* **Herencia**: evitar repetir código y crear jerarquías claras.
* **Polimorfismo**: trabajar con distintos objetos de forma uniforme.
