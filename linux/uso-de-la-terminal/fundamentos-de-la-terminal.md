---
icon: terminal
---

# Fundamentos de la terminal

### Qué es una shell

> Una shell es el intérprete de comandos que se sitúa entre el usuario y el kernel. Recibe lo que escribes, lo interpreta y ejecuta los programas correspondientes.

#### Terminal vs shell vs consola

| Término  | Qué es                                                                                                     |
| -------- | ---------------------------------------------------------------------------------------------------------- |
| Terminal | El programa que muestra la ventana donde escribes (xterm, GNOME Terminal, Terminator, Windows Terminal...) |
| Shell    | El intérprete de comandos que corre dentro de la terminal (bash, zsh, sh...)                               |
| Consola  | Acceso físico/directo al sistema, sin capa gráfica (ej. `tty1`, consola de un hipervisor)                  |

La terminal es la "ventana", la shell es el "programa que interpreta lo que escribes dentro de esa ventana".

#### Tipos de shell

| Shell  | Dónde se usa                               | Notas                                              |
| ------ | ------------------------------------------ | -------------------------------------------------- |
| `bash` | Por defecto en la mayoría de distros Linux | La más común, la que usarás en el 90% de los casos |
| `sh`   | POSIX shell, scripts portables             | Subconjunto mínimo, sin arrays ni `[[ ]]`          |
| `zsh`  | macOS por defecto, Kali opcional           | Más features (autocompletado, plugins, oh-my-zsh)  |
| `dash` | `/bin/sh` en Debian/Ubuntu                 | Más rápida, menos features que bash                |

```bash
echo $SHELL      # shell configurada para el usuario
echo $0            # shell que se está ejecutando ahora mismo
cat /etc/shells     # shells instaladas en el sistema
chsh -s /bin/zsh     # cambiar la shell por defecto del usuario
```

### El prompt

Formato típico: `usuario@host:directorio$`. El `$` indica usuario normal, `#` indica root.

```
pyrhos@debian:~/Documentos$
```

### PATH y variables de entorno

| Variable | Contenido                                     |
| -------- | --------------------------------------------- |
| `$HOME`  | Directorio home del usuario                   |
| `$USER`  | Usuario actual                                |
| `$PATH`  | Rutas donde el sistema busca ejecutables      |
| `$PWD`   | Directorio de trabajo actual                  |
| `$SHELL` | Shell por defecto del usuario                 |
| `$?`     | Código de salida del último comando ejecutado |
| `$$`     | PID del proceso de la shell actual            |

```bash
echo $PATH
export MI_VARIABLE="valor"      # crear variable de entorno
env                              # listar todas las variables de entorno
printenv PATH                    # ver una variable concreta
```

`export` hace que la variable esté disponible para procesos hijos. Una variable normal (`VAR=valor`) solo existe en la shell actual.

#### Resolución de comandos vía PATH

Cuando escribes un comando, la shell busca un ejecutable con ese nombre en cada directorio listado en `$PATH`, en orden.

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

which nmap          # ruta del ejecutable que se usará
type cd               # indica si es builtin, alias o ejecutable

export PATH=$PATH:/ruta/nueva    # añadir un directorio (temporal, solo sesión actual)
```

### Atajos de teclado esenciales

| Atajo    | Acción                                             |
| -------- | -------------------------------------------------- |
| `Ctrl+C` | Interrumpe el proceso en ejecución (SIGINT)        |
| `Ctrl+Z` | Suspende el proceso (lo manda a background parado) |
| `Ctrl+D` | Cierra la sesión / EOF                             |
| `Ctrl+L` | Limpia la pantalla (igual que `clear`)             |
| `Ctrl+A` | Cursor al inicio de la línea                       |
| `Ctrl+E` | Cursor al final de la línea                        |
| `Ctrl+U` | Borra desde el cursor hasta el inicio              |
| `Ctrl+K` | Borra desde el cursor hasta el final               |
| `Ctrl+W` | Borra la palabra anterior                          |
| `Ctrl+R` | Búsqueda inversa en el historial                   |

### Autocompletado con Tab

```
cd /var/l<Tab>        # completa a /var/log/ si no hay ambigüedad
ls /var/<Tab><Tab>     # doble Tab: muestra todas las opciones posibles
```

Funciona también con nombres de comandos, variables (`$HO<Tab>` → `$HOME`) y opciones de muchos programas si tienes `bash-completion` instalado.

### Historial

```bash
history               # ver historial completo
history 20             # últimos 20 comandos
!!                      # repetir el último comando
!n                      # repetir el comando número n del historial
!cadena                 # repetir el último comando que empieza por "cadena"
Ctrl+R                  # búsqueda interactiva en el historial
```

El historial se guarda en `~/.bash_history` (o `~/.zsh_history` en zsh) al cerrar la sesión.

### Alias

```bash
alias ll='ls -la'
alias ..='cd ..'
unalias ll             # eliminar un alias
alias                   # ver alias activos
```

Para que un alias persista, añádelo a `~/.bashrc` o `~/.zshrc` y recarga con `source ~/.bashrc`.
