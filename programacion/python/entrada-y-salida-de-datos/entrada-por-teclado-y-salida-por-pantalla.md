---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/z0iyBve0fcLSJpEvb9YL/analisis-de-trafico/wireshark/filtros-de-wireshark
---

# Entrada por teclado y salida por pantalla

En Python para interactuar con el usuario se hace principalmente con:

* `input()` → leer datos del teclado
* `print()` → mostrar información en pantalla

```python
# Leer datos del usuario
nombre = input("Introduce tu nombre: ")
print("Hola,", nombre)
```

***

### Formato de Texto

Podemos mostrar y organizar la información de manera clara usando:

* Concatenación de cadenas
* Formateo con f-strings (recomendado)
* Métodos como `.format()`

```python
edad = 20
print(f"Hola {nombre}, tienes {edad} años")
print("Hola {}, tienes {} años".format(nombre, edad))
```

También se puede alinear texto o rellenar con caracteres:

```python
print(f"{'Titulo':^20}")  # centrado en 20 caracteres
print(f"{'Nombre':<10} {'Edad':>5}")  # izquierda y derecha
```

***

### Codificación de Caracteres

Python maneja strings como Unicode por defecto. Esto asegura que se puedan mostrar y leer caracteres de distintos idiomas sin problemas.

```python
mensaje = "¡Hola, mundo! Привет мир! こんにちは世界！"
print(mensaje)
```

***

