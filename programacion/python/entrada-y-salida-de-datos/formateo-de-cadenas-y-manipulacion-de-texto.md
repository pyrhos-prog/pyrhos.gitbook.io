# Formateo de cadenas y manipulación de texto

Python ofrece varias formas de formatear y trabajar con cadenas de texto.

### Formateo de Cadenas

#### 1. Formateo Clásico (`%`)

Similar a `printf` en C.

```python
nombre = "Ana"
edad = 25
print("Hola %s, tienes %d años" % (nombre, edad))
```

#### 2. Método `format()`

&#x20;Permite alinear, rellenar y trabajar con números.

```python
print("Hola {}, tienes {} años".format(nombre, edad))
print("Nombre: {:<10} Edad: {:>3}".format(nombre, edad))  # alineación
```

#### 3. F-strings&#x20;

Es la forma mas simple de todas.

```python
print(f"Hola {nombre}, tienes {edad} años")
print(f"{nombre:^10} | {edad:>3}")  # centrado y alineado a la derecha
```

### Manipulación de Texto

#### Búsqueda y reemplazo

```python
texto = "Hola Mundo"
print(texto.find("Mundo"))  # posición
print(texto.replace("Mundo", "Python"))
```

#### Métodos de prueba

```python
s = "1234"
print(s.isdigit())   # True
print(s.isalpha())   # False
print(s.startswith("1"))  # True
print(s.endswith("4"))    # True
```

#### Transformaciones

```python
texto = "hola A TODO"
print(texto.upper())    # HOLA A TODOS
print(texto.lower())    # hola a todos 
print(texto.split())    # ['hola', 'a', 'TODOS']
print(" ".join(["Hola", "Python"]))  # Hola Python
```

#### Unicode

Python maneja cadenas Unicode por defecto, lo que permite soportar caracteres de distintos idiomas.

```python
mensaje = "¡Hola! Привет! こんにちは!"
print(mensaje)
```
