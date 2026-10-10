---
icon: terminal
---

# Edición en terminal

### nano básico

Editor simple, ideal para ediciones rápidas sin curva de aprendizaje.

```bash
nano fichero.txt
```

| Atajo    | Acción              |
| -------- | ------------------- |
| `Ctrl+O` | Guardar (Write Out) |
| `Ctrl+X` | Salir               |
| `Ctrl+K` | Cortar línea        |
| `Ctrl+U` | Pegar               |
| `Ctrl+W` | Buscar              |
| `Ctrl+\` | Buscar y reemplazar |
| `Ctrl+G` | Ayuda               |

Los atajos se muestran siempre en la barra inferior de nano, no hace falta memorizarlos todos.

### vim: modos y comandos&#x20;

Vim tiene varios modos; el error más común es intentar escribir estando en modo normal.

| Modo      | Cómo entrar                       | Para qué                          |
| --------- | --------------------------------- | --------------------------------- |
| Normal    | `Esc` (modo por defecto al abrir) | Navegar, borrar, copiar, comandos |
| Inserción | `i`, `a`, `o` desde normal        | Escribir texto                    |
| Visual    | `v` desde normal                  | Seleccionar texto                 |
| Comando   | `:` desde normal                  | Guardar, salir, buscar/reemplazar |

```bash
vim fichero.txt
```

#### Entrar en modo inserción

| Tecla | Efecto                                         |
| ----- | ---------------------------------------------- |
| `i`   | Inserta antes del cursor                       |
| `a`   | Inserta después del cursor                     |
| `o`   | Nueva línea debajo, entra en inserción         |
| `O`   | Nueva línea encima, entra en inserción         |
| `Esc` | Vuelve a modo normal desde cualquier otro modo |

#### Guardar y salir

```
:w          guardar
:q          salir (falla si hay cambios sin guardar)
:wq         guardar y salir
:q!         salir SIN guardar, descarta cambios
:x          guardar y salir (equivalente a :wq si hay cambios)
```

#### Navegación básica en modo normal

| Tecla     | Acción                                |
| --------- | ------------------------------------- |
| `h j k l` | Izquierda, abajo, arriba, derecha     |
| `gg`      | Ir al inicio del fichero              |
| `G`       | Ir al final del fichero               |
| `0`       | Inicio de línea                       |
| `$`       | Final de línea                        |
| `/patrón` | Buscar hacia adelante                 |
| `n`       | Siguiente coincidencia de la búsqueda |

#### Borrar, copiar, pegar

| Comando  | Acción                           |
| -------- | -------------------------------- |
| `x`      | Borra el carácter bajo el cursor |
| `dd`     | Borra (corta) la línea actual    |
| `yy`     | Copia la línea actual            |
| `p`      | Pega después del cursor/línea    |
| `u`      | Deshacer                         |
| `Ctrl+R` | Rehacer                          |

#### Buscar y reemplazar

```
:%s/antiguo/nuevo/g        reemplaza en todo el fichero
:s/antiguo/nuevo/g          reemplaza solo en la línea actual
:%s/antiguo/nuevo/gc         reemplaza con confirmación por cada coincidencia
```

{% hint style="info" %}
Para sobrevivir en vim lo mínimo imprescindible es: `i` para escribir, `Esc` para salir de inserción, `:wq` para guardar y salir, `:q!` si todo sale mal y quieres cerrar sin guardar nada.
{% endhint %}

### cat, less, head, tail

```bash
cat fichero.txt            # vuelca todo el contenido por pantalla
cat -n fichero.txt           # con número de línea

less fichero.txt              # paginador: navegación con flechas, / para buscar, q para salir
head fichero.txt                # primeras 10 líneas
head -n 20 fichero.txt            # primeras 20 líneas
tail fichero.txt                   # últimas 10 líneas
tail -n 20 fichero.txt               # últimas 20 líneas

tail -f fichero.log                    # sigue el fichero en tiempo real, muy usado con logs
tail -f /var/log/auth.log | grep "Failed"   # seguir un log filtrando en vivo
```

| Comando   | Cuándo usarlo                                               |
| --------- | ----------------------------------------------------------- |
| `cat`     | Ficheros cortos, o para enviarlos por una tubería           |
| `less`    | Ficheros largos que quieres explorar con calma              |
| `head`    | Ver el principio (cabeceras CSV, primeras líneas de un log) |
| `tail -f` | Monitorizar un log en tiempo real                           |

