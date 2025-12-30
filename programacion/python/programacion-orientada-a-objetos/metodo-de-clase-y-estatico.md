---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/z0iyBve0fcLSJpEvb9YL/protocolos-de-transmision/protocolo-udp
---

# Método de clase y estático

#### Métodos de clase (`@classmethod`)

Un método de clase trabaja con la clase en general, no con un objeto concreto. Se define con `@classmethod` y usa `cls`.

```python
class Usuario:
    tipo = "normal"

    @classmethod
    def cambiar_tipo(cls, nuevo_tipo):
        cls.tipo = nuevo_tipo

Usuario.cambiar_tipo("admin")
print(Usuario.tipo)
```

Se usa cuando el método tiene sentido para toda la clase.

#### Métodos estáticos (`@staticmethod`)

Un método estático no usa ni `self` ni `cls`. Es una función normal que se coloca dentro de la clase por organización.

```python
class Calculadora:
    @staticmethod
    def sumar(a, b):
        return a + b

print(Calculadora.sumar(2, 3))
```
