# Introducción

> Conceptos básicos para empezar a trabajar con Bash: qué es, cómo se ejecuta un script, permisos, convenciones y dónde buscar ayuda.

### Qué es Bash

Bash (Bourne Again SHell) es un intérprete de comandos y a la vez un lenguaje de scripting. Es la shell por defecto en la mayoría de distribuciones Linux.

Permite:

* Ejecutar comandos de forma interactiva.
* Automatizar tareas agrupando comandos en scripts.
* Usar variables, condicionales, bucles y funciones.

```bash
echo $SHELL          # shell por defecto del usuario
bash --version       # versión instalada
```

Otras shells habituales:

| Shell  | Notas                                                  |
| ------ | ------------------------------------------------------ |
| `sh`   | Shell POSIX básica. En Debian/Ubuntu suele ser `dash`. |
| `zsh`  | Muy extendida en uso interactivo.                      |
| `fish` | Orientada a la comodidad, sintaxis distinta.           |
| `dash` | Ligera y rápida, solo sintaxis POSIX.                  |

Un script escrito con características propias de Bash (arrays, `[[ ]]`, etc.) no funciona en `sh`/`dash`.

### Terminal, shell e intérprete

Son tres conceptos distintos que suelen confundirse.

| Concepto   | Qué es                                                                                |
| ---------- | ------------------------------------------------------------------------------------- |
| Terminal   | Programa (emulador) que muestra una ventana de texto donde se escribe y se lee.       |
| Shell      | Programa que recibe lo que se escribe y lo ejecuta. Bash es una shell.                |
| Intérprete | Forma de trabajar de la shell: lee y ejecuta las órdenes línea a línea, sin compilar. |

Flujo básico: el usuario escribe en la terminal, la shell interpreta la línea y ejecuta el comando, y la salida vuelve a la terminal.

#### Tipos de comandos

```bash
type cd        # cd is a shell builtin
type ls        # ls is /usr/bin/ls
type ll        # ll is aliased to 'ls -l'
```

| Tipo    | Descripción                                                   |
| ------- | ------------------------------------------------------------- |
| Builtin | Integrado en Bash (`cd`, `echo`, `read`, `export`).           |
| Externo | Programa en disco, localizado mediante `PATH` (`ls`, `grep`). |
| Alias   | Atajo definido por el usuario.                                |
| Función | Código definido en la propia shell o script.                  |

### Primer script

#### Estructura mínima

```bash
#!/bin/bash

echo "Hola, mundo"
```

* La primera línea (`#!`) se llama **shebang** e indica qué intérprete ejecuta el fichero.
* Alternativa más portable: `#!/usr/bin/env bash`, que busca Bash en el `PATH`.
* La extensión `.sh` es convención, no obligatoria.

#### Formas de ejecutarlo

```bash
chmod +x script.sh     # dar permiso de ejecución (una vez)
./script.sh            # ejecutar como programa

bash script.sh         # ejecutar pasando el fichero a Bash (no necesita permiso x)
source script.sh       # ejecutar en la shell actual (también: . script.sh)
```

| Forma              | Proceso          | Uso típico                                |
| ------------------ | ---------------- | ----------------------------------------- |
| `./script.sh`      | Nuevo (subshell) | Ejecutar el script normalmente            |
| `bash script.sh`   | Nuevo            | Probar sin permisos de ejecución          |
| `source script.sh` | Shell actual     | Cargar variables y funciones en la sesión |

Las variables definidas en un script ejecutado en un proceso nuevo desaparecen al terminar. Con `source` permanecen.

### Permisos de ejecución

Para ejecutar un fichero con `./script.sh` necesita el bit de ejecución.

```bash
ls -l script.sh
# -rw-r--r-- 1 usuario usuario 35 ene  1 10:00 script.sh   (sin permiso x)

chmod +x script.sh
ls -l script.sh
# -rwxr-xr-x 1 usuario usuario 35 ene  1 10:00 script.sh
```

Errores típicos:

| Mensaje                                      | Causa                                                                                         |
| -------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `Permission denied`                          | Falta el bit de ejecución.                                                                    |
| `command not found`                          | Se escribió `script.sh` en lugar de `./script.sh`; el directorio actual no está en el `PATH`. |
| `bad interpreter: No such file or directory` | Shebang incorrecto o fin de línea de Windows (CRLF). Se corrige con `dos2unix script.sh`.     |

Los permisos se tratan en detalle en el tema _Shell y sistema de ficheros_.

### Comentarios y convenciones

#### Comentarios

```bash
# Comentario de línea completa
echo "Hola"   # Comentario al final de línea
```

No existe comentario de bloque nativo. Se usa `#` en cada línea.

#### Cabecera recomendada

```bash
#!/bin/bash
#
# nombre.sh - Descripción breve de lo que hace
# Uso: ./nombre.sh [opciones]
```

#### Convenciones de estilo

| Aspecto                           | Recomendación                              |
| --------------------------------- | ------------------------------------------ |
| Nombres de variables              | Minúsculas con guion bajo: `ruta_destino`  |
| Constantes y variables de entorno | Mayúsculas: `MAX_INTENTOS`, `PATH`         |
| Nombres de funciones              | Minúsculas con guion bajo: `mostrar_ayuda` |
| Indentación                       | 2 o 4 espacios, siempre la misma           |
| Ficheros                          | Minúsculas, sin espacios, extensión `.sh`  |
| Variables                         | Entrecomilladas al usarlas: `"$variable"`  |

### Cómo obtener ayuda

| Comando                                | Para qué sirve                                  |
| -------------------------------------- | ----------------------------------------------- |
| `man comando`                          | Manual completo de un programa externo.         |
| `help comando`                         | Ayuda de un builtin de Bash (`help cd`).        |
| `comando --help`                       | Resumen de uso rápido de muchos programas.      |
| `type comando`                         | Indica si es builtin, alias, función o fichero. |
| `which comando` / `command -v comando` | Ruta del ejecutable.                            |
| `whatis comando`                       | Descripción de una línea.                       |
| `apropos palabra`                      | Busca manuales relacionados con una palabra.    |
| `info comando`                         | Documentación GNU en formato info.              |

```bash
man grep
help read
ls --help | less
apropos "copy files"
man 5 crontab        # el número indica la sección del manual
```

Secciones del manual más usadas: `1` comandos, `5` formatos de ficheros, `7` conceptos varios, `8` administración.

En `man`: `/texto` busca, `n` siguiente coincidencia, `q` sale.

La ayuda propia de Bash está en `man bash`, útil para consultar la sintaxis exacta de expansiones y operadores.
