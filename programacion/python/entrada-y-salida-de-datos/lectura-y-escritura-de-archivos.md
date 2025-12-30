# Lectura y escritura de archivos

En Python trabajar con archivos se hace principalmente con la función `open()`. Esta crea un objeto archivo que podemos leer o modificar según el **modo de apertura**:

* `'r'` → lectura (por defecto)
* `'w'` → escritura (sobrescribe si el archivo existe)
* `'a'` → añadir al final del archivo
* `'b'` → modo binario (leer/escribir bytes)

```python
# Abrir un archivo en modo lectura
archivo = open("datos.txt", "r")
contenido = archivo.read()
archivo.close()
print(contenido)
```

### Lectura de Archivos

Formas de leer un archivo:

* `read()` → lee todo el contenido de golpe
* `readline()` → lee una línea a la vez
* Iterar sobre el archivo → línea por línea

```python
# Leer línea por línea
with open("datos.txt", "r") as archivo:
    for linea in archivo:
        print(linea.strip())
```

### Escritura en Archivos

Formas de escribir:

* `write()` → escribe una cadena
* `writelines()` → escribe una lista de cadenas

```python
# Escribir en un archivo
with open("salida.txt", "w") as archivo:
    archivo.write("Hola Mundo\n")
    archivo.writelines(["Linea 1\n", "Linea 2\n"])
```

> Nota: `'w'` borra el contenido existente, `'a'` añade al final.

### Manejadores de Contexto

Usar `with open()` asegura que el archivo se cierre automáticamente al terminar el bloque, incluso si ocurre un error. Esto previene problemas como fugas de recursos.

```python
# Forma recomendada de abrir archivos
with open("datos.txt", "r") as archivo:
    contenido = archivo.read()
    print(contenido)
# No hace falta llamar a close()
```

