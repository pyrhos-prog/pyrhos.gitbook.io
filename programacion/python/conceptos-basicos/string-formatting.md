# String formatting

### 1. Operador `%` (Porcentaje)

* Método clásico, también llamado **interpolación de cadenas**.
* Usa **marcadores de posición**:
  * `%s` → cadena (`str`)
  * `%d` → entero (`int`)
  * `%f` → número flotante (`float`)

**Ejemplo:**

```python
nombre = "Ana"
edad = 25
print("Me llamo %s y tengo %d años" % (nombre, edad))
```

**Salida:**

```
Me llamo Ana y tengo 25 años
```

***

### 2. Método `format()`

* Introducido en **Python 2.6**.
* Usa **llaves `{}`** como marcadores de posición.
* Más flexible y legible que `%`.

**Ejemplo:**

```python
nombre = "Ana"
edad = 25
print("Me llamo {} y tengo {} años".format(nombre, edad))
```

**Salida:**

```
Me llamo Ana y tengo 25 años
```

* También permite **posiciones y nombres**:

```python
print("Edad: {1}, Nombre: {0}".format(nombre, edad))
print("Nombre: {nombre}, Edad: {edad}".format(nombre="Ana", edad=25))
```

***

### 3. F-Strings (Literal String Interpolation)

* Disponible desde **Python 3.6**.
* Forma **concisa y legible** de insertar variables.
* Se antepone una **`f`** a la cadena y se usan **llaves `{}`** para las variables o expresiones.

**Ejemplo:**

```python
nombre = "Ana"
edad = 25
print(f"Me llamo {nombre} y tengo {edad} años")
```

**Salida:**

```
Me llamo Ana y tengo 25 años
```

* Permite incluso **operaciones dentro de las llaves**:

```python
print(f"El doble de mi edad es {edad * 2}")
```
