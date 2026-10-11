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

Lo visto antes (`$HOME`, `$PATH`, `export`) solo dura mientras la sesión actual está abierta. Para que una variable o un cambio en el `PATH` persista entre sesiones, hay que escribirlo en el fichero de inicio correcto.

#### Qué fichero se carga y cuándo

| Fichero            | Cuándo se carga                                                                |
| ------------------ | ------------------------------------------------------------------------------ |
| `/etc/environment` | Al iniciar sesión, a nivel de todo el sistema (todos los usuarios)             |
| `/etc/profile`     | Shells de login, a nivel de sistema                                            |
| `~/.profile`       | Shells de login del usuario (independiente de bash/zsh)                        |
| `~/.bashrc`        | Cada nueva shell interactiva de bash (lo más común para alias y PATH personal) |
| `~/.bash_profile`  | Shell de login en bash — si existe, sustituye a `~/.profile` para bash         |
| `~/.zshrc`         | Equivalente a `.bashrc` pero para zsh                                          |

{% hint style="info" %}
"Shell de login" es la que se abre al iniciar sesión (ej. consola física, `ssh usuario@host`). Una terminal gráfica nueva dentro de un escritorio ya logueado normalmente abre una shell no-login, interactiva — por eso `.bashrc` es el sitio más fiable para persistir cosas: se carga en casi todos los casos.
{% endhint %}

#### Hacer persistente una variable o el PATH

```bash
# Añadir al final de ~/.bashrc
echo 'export PATH=$PATH:/opt/herramientas/bin' >> ~/.bashrc
echo 'export EDITOR=vim' >> ~/.bashrc

# Aplicar sin cerrar sesión
source ~/.bashrc
```

#### A nivel de todo el sistema (requiere privilegios)

```bash
# /etc/environment usa formato KEY="value", sin "export" y sin referenciar otras variables
echo 'MI_VARIABLE="valor"' | sudo tee -a /etc/environment
```

#### Comprobar el origen de una variable o función

```bash
type -a nombre          # muestra todas las definiciones (builtin, alias, función, ejecutable)
declare -p PATH            # muestra cómo está declarada una variable
set | grep NOMBRE            # buscar una variable entre todas las definidas en la sesión actual
```

#### Orden de carga en una shell de login interactiva (bash)

```
/etc/profile → ~/.bash_profile (o ~/.bash_login o ~/.profile, el primero que exista) → ~/.bashrc (si se invoca desde .bash_profile)
```

{% hint style="info" %}
Si defines la misma variable en varios ficheros, gana la que se carga último. Es una fuente habitual de confusión cuando "algo que configuré no funciona" — comprobar siempre con `echo $VARIABLE` tras un `source` para confirmar qué valor quedó activo.
{% endhint %}

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

