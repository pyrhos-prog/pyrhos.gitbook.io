---
icon: terminal
---

# Control del flujo stdout, stdin y stderr

### 1. Concepto básico

Cada proceso en Linux arranca con **3 canales de comunicación** abiertos, identificados por un número llamado _file descriptor_ (FD):

| FD  | Nombre     | Función           | Por defecto |
| --- | ---------- | ----------------- | ----------- |
| `0` | **stdin**  | Entrada de datos  | Teclado     |
| `1` | **stdout** | Salida normal     | Pantalla    |
| `2` | **stderr** | Salida de errores | Pantalla    |

stdout y stderr salen ambos por pantalla, pero son **canales distintos**. Por eso se pueden redirigir por separado.

#### 2. Ejemplos de cada flujo

**stdout**: resultado normal de un comando.

bash

```bash
echo "Hola, mundo!"
```

**stdin**: datos que recibe el programa, ya sea del teclado, de un archivo o de otro comando.

bash

```bash
read -p "Ingresa tu nombre: " nombre
```

**stderr**: mensajes de error.

bash

```bash
ls archivo_inexistente
# ls: no se puede acceder a 'archivo_inexistente': No existe el archivo o el directorio
```

#### 3. Operadores de redirección

| Operador | Qué hace                                             | Ejemplo                      |
| -------- | ---------------------------------------------------- | ---------------------------- |
| `>`      | Redirige stdout a un archivo (**sobrescribe**)       | `ls > lista.txt`             |
| `>>`     | Redirige stdout a un archivo (**añade al final**)    | `ls >> lista.txt`            |
| `2>`     | Redirige stderr a un archivo                         | `ls noexiste 2> errores.txt` |
| `2>&1`   | Envía stderr al mismo sitio que stdout               | `cmd > todo.txt 2>&1`        |
| `<`      | Usa un archivo como stdin                            | `sort < nombres.txt`         |
| `\|`     | Conecta el stdout de un comando con el stdin de otro | `ls \| grep ".txt"`          |

#### 4. /dev/null: el agujero negro

`/dev/null` es un archivo especial que **descarta todo lo que se escribe en él**. Sirve para silenciar salidas.

**Ocultar solo errores:**

bash

```bash
ls archivo_inexistente 2>/dev/null
```

**Ocultar solo la salida normal:**

bash

```bash
ls >/dev/null
```

**Ocultar todo (stdout + stderr):**

bash

```bash
comando >/dev/null 2>&1
```

Equivalente en Bash (más corto, pero no funciona en `sh` puro):

bash

```bash
comando &>/dev/null
```

#### 5. Ojo con el orden: `>/dev/null 2>&1`

Las redirecciones se leen **de izquierda a derecha**:

bash

```bash
comando >/dev/null 2>&1   # Correcto: stdout -> null, y stderr -> donde apunta stdout (null)
comando 2>&1 >/dev/null   # Distinto: stderr -> pantalla (donde apuntaba stdout), stdout -> null
```

En el segundo caso los errores **siguen saliendo por pantalla**. `2>&1` significa “copia el destino actual de stdout”, así que importa qué destino tenga en ese momento.

#### 6. Ejemplos prácticos

**Buscar archivos sin ruido de “Permiso denegado”:**

bash

```bash
find / -name "*.conf" 2>/dev/null
```

**Guardar la salida y los errores en archivos separados:**

bash

```bash
nmap 10.10.10.5 > scan.txt 2> errores.txt
```

**Guardar todo en un único log:**

bash

```bash
./script.sh > log.txt 2>&1
```

**Ver la salida por pantalla y guardarla a la vez:**

bash

```bash
nmap 10.10.10.5 | tee scan.txt
```

**Ejecutar en segundo plano sin generar salida:**

bash

```bash
./script.sh >/dev/null 2>&1 &
```

#### Resumen rápido
