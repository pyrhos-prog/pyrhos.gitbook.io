# Variables y tipos de datos

#### 1. Números

**a) Enteros (`int`)**

* Son **números sin decimales**.
* Ejemplo: `5`, `-10`, `100`
* Usos: contar cosas, índices, operaciones matemáticas simples.

**b) Flotantes (`float`)**

* Son **números con decimales**.
* Ejemplo: `3.14`, `-0.5`, `100.0`
* Usos: medir cantidades más precisas, cálculos con decimales.

***

#### 2. Texto (`str`)

* Una **cadena de caracteres**, como palabras o frases.
* Siempre va entre **comillas simples `' '`** o **comillas dobles `" "`**.
* Ejemplo: `"Hola"`, `'Python'`, `"123"`
* Importante: los strings son **inmutables**, no se puede cambiar un carácter individual sin crear uno nuevo.
* Usos: nombres, mensajes, etiquetas, cualquier texto.

***

#### 3. Booleanos&#x20;

* Solo pueden ser **True (verdadero)** o **False (falso)**.
* Ejemplo: `is_raining = True`
* Usos: decisiones en el programa, condicionales (`if`), encender/apagar cosas.

***

#### 4. Listas&#x20;

* Una **colección de elementos ordenados** que **pueden cambiar**.
* Pueden contener **diferentes tipos de datos** juntos.
* Ejemplo: `[1, 2, 3]`, `[1, "Hola", 3.5]`
* Accedemos a los elementos usando su **posición** (empezando desde 0).
* Usos: guardar varias cosas relacionadas, como nombres de alumnos, notas, productos.

***

#### 5. Tuplas&#x20;

* Similar a una lista, pero **no se puede cambiar** después de crearla.
* Ejemplo: `(1, 2, 3)`
* Usos: datos que **no deben modificarse**, como coordenadas o constantes.

***

#### 6. Conjuntos&#x20;

* Colección de elementos **únicos** y **sin orden específico**.
* Ejemplo: `{1, 2, 3}`
* No permite duplicados: `{1, 2, 2, 3}` → `{1, 2, 3}`
* Usos: eliminar duplicados, operaciones matemáticas (intersección, unión).

***

#### 7. Diccionarios&#x20;

* Colección de **pares clave:valor**, como un pequeño mapa.
* Ejemplo: `{"nombre": "Ana", "edad": 25}`
* Accedemos a los datos usando la **clave**, no la posición.
* Usos: guardar información relacionada, como datos de una persona, configuración, resultados.

***

| Tipo  | Qué guarda                   | Ejemplo                       |
| ----- | ---------------------------- | ----------------------------- |
| int   | Números sin decimales        | 5                             |
| float | Números con decimales        | 3.14                          |
| str   | Texto                        | "Hola"                        |
| bool  | Verdadero/Falso              | True                          |
| list  | Colección ordenada mutable   | \[1, "Hola", 3.5]             |
| tuple | Colección ordenada inmutable | (1, 2, 3)                     |
| set   | Conjunto único, sin orden    | {1, 2, 3}                     |
| dict  | Pares clave:valor            | {"nombre": "Ana", "edad": 25} |
