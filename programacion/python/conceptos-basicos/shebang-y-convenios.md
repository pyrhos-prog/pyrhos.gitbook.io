---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/z0iyBve0fcLSJpEvb9YL/modelos-de-referencia/modelo-tcp-ip
---

# Shebang y convenios

Son aspectos importantes que facilitan la escritura de scripts claros y portables

### 1. Shebang en Python

#### ¿Qué es el shebang?

Es la **primera línea** de un script ejecutable que indica **qué intérprete** debe usar el sistema operativo.

```
#!/usr/bin/env python3
```

#### ¿Cómo funciona?

```
┌───────────────┐
│ Script Python │
│  ejecutable   │
└───────┬───────┘
        │
        ▼
┌────────────────────┐
│ Sistema Operativo  │
└───────┬────────────┘
        │ lee el shebang
        ▼
┌────────────────────┐
│ /usr/bin/env       │
│ busca python3      │
└───────┬────────────┘
        ▼
┌────────────────────┐
│ Intérprete Python3 │
└────────────────────┘
```

#### ¿Por qué usar `/usr/bin/env`?

* Usa el **Python 3 del entorno activo**
* Evita rutas absolutas
* Mejora la **portabilidad** entre sistemas

***

### 2. Convenios de Codificación en Python (PEP 8)

#### ¿Qué es PEP 8?

Es la **guía oficial de estilo** para escribir código Python claro, legible y consistente.

***

#### 2.1 Nombres (Naming)

| Elemento   | Convención                    | Ejemplo           |
| ---------- | ----------------------------- | ----------------- |
| Variables  | `lower_case_with_underscores` | `total_count`     |
| Funciones  | `lower_case_with_underscores` | `calculate_sum()` |
| Constantes | `UPPER_CASE_WITH_UNDERSCORES` | `MAX_SIZE`        |
| Clases     | `CamelCase`                   | `UserProfile`     |

***

#### 2.2 Longitud de línea

```
Código:       máximo 79 caracteres
Comentarios:  máximo 72 caracteres
```

✔ Mejora la lectura en editores y revisiones de código.

***

#### 2.3 Indentación

* **4 espacios por nivel**
* No usar tabuladores

Ejemplo correcto:

```python
if condition:
    do_something()
    if other_condition:
        do_other_thing()
```

***

#### 2.4 Espacios en blanco

Correcto:

```python
func(a, b)
my_list = [1, 2, 3]
```

Incorrecto:

```python
func( a , b )
my_list = [ 1 , 2 , 3 ]
```

Reglas clave:

* Un espacio después de comas
* No espacios innecesarios dentro de paréntesis

***

#### 2.5 Importaciones

Orden recomendado:

```
1. Biblioteca estándar
2. Módulos de terceros
3. Módulos locales
```

Ejemplo:

```python
import os
import sys

import requests

from my_project import utils
```

* Una importación por línea
* Separadas por líneas en blanco

***

### 3. Resumen Visual Rápido

```
Shebang  → define intérprete
PEP 8    → define estilo
```

```
Código claro
    ↓
Más legible
    ↓
Más mantenible
    ↓
Menos errores
```
