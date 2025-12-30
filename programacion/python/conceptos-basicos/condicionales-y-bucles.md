# Condicionales y bucles

### Condicionales

* if: Si una condición es verdadera ejecuta un bloque de código.

```python
edad = 10
if edad <= 18:
    print("Eres menor de edad")
```

* else: Si la condición if o elif no se ha cumplido esta ejecuta un bloque de código.

```python
edad = 34
if edad <= 18:
    print("Eres menor de edad")
else:
    print("Eres mayor de edad")
```

* elif: Se utiliza para verificar múltiples expresiones sólo si las anteriores no son verdaderas.

```python
edad = 20
if edad < 13:
print("Eres un niño")
elif edad < 20:
print("Eres un adolescente")
else:
print("Eres un adulto")
```



## Bucles

#### for

```python
for i in range(5):
    print(i)
```

o con una lista

```python
lista_nombres=["Jesus", "Samuel", "Adrian", "Sergio"]

for nombre in lista_nombres:
    print(nombre)
```

#### while

bucle infinito:

```python
numero = 1
while numero < 5:
    print(numero)
```

bucle que aumenta el valor en 1 por cada vez:

```python
numero = 1
while numero < 5: 
    print(numero)
    numero +=1
```

### Control de flujo en bucles

* **break**: Termina el bucle y pasa el control a la siguiente declaración fuera del bucle.
* **continue**: Omite el resto del código dentro del bucle y continúa con la siguiente iteración.
* **pass**: No hace nada, se utiliza como una declaración de relleno donde el código eventualmente irá, pero no ha sido escrito todavía.
