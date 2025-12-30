# Encapsulamiento

El encapsulamiento sirve para controlar cómo se accede a los datos de una clase.\
En Python no se fuerza el acceso. Se usan **convenciones de nombres**.

### Niveles de visibilidad

| Tipo      | Prefijo | Explicación                                          |
| --------- | ------- | ---------------------------------------------------- |
| Público   | ninguno | Se puede usar desde cualquier parte del programa     |
| Protegido | `_`     | Indica que es de uso interno o para subclases        |
| Privado   | `__`    | Python cambia el nombre internamente (name mangling) |

***

#### Ejemplo de atributos

```python
class Usuario:
    def __init__(self):
        self.nombre = "Ana"       # público
        self._edad = 20           # protegido
        self.__password = "1234"  # privado

u = Usuario()
print(u.nombre)
print(u._edad)
# print(u.__password)  # Error
```

* Los públicos forman parte normal de la clase.
* Los protegidos se pueden usar, pero no es buena práctica.
* Los privados evitan accesos accidentales desde fuera.
