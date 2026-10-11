---
icon: terminal
---

# Gestión de usuarios y grupos

### Ficheros clave

| Fichero        | Contenido                                                                  |
| -------------- | -------------------------------------------------------------------------- |
| `/etc/passwd`  | Lista de usuarios: nombre, UID, GID, home, shell (legible por todos)       |
| `/etc/shadow`  | Contraseñas (hasheadas) y políticas de expiración — solo root puede leerlo |
| `/etc/group`   | Lista de grupos y sus miembros                                             |
| `/etc/sudoers` | Quién puede usar `sudo` y con qué privilegios                              |

```bash
cat /etc/passwd
```

```
pyrhos:x:1000:1000:Osman:/home/pyrhos:/bin/bash
```

| Campo          | Significado                                                         |
| -------------- | ------------------------------------------------------------------- |
| `pyrhos`       | Nombre de usuario                                                   |
| `x`            | Contraseña (la real está en `/etc/shadow`, `x` es solo un marcador) |
| `1000`         | UID                                                                 |
| `1000`         | GID principal                                                       |
| `Osman`        | Comentario/nombre completo (GECOS)                                  |
| `/home/pyrhos` | Directorio home                                                     |
| `/bin/bash`    | Shell asignada                                                      |

{% hint style="info" %}
Un usuario con shell `/usr/sbin/nologin` o `/bin/false` no puede iniciar sesión interactiva — típico de cuentas de servicio (www-data, mysql...). Verlo en `/etc/passwd` ayuda a distinguir usuarios "reales" de cuentas de sistema.
{% endhint %}

### Crear, modificar y borrar usuarios

```bash
sudo useradd -m -s /bin/bash nuevo_usuario     # -m crea el home, -s fija la shell
sudo useradd -m -G sudo,docker nuevo_usuario     # añade a grupos secundarios al crearlo

sudo passwd nuevo_usuario         # establecer/cambiar contraseña

sudo usermod -aG docker usuario      # añadir a un grupo adicional (-a = append, importante no olvidarlo)
sudo usermod -s /bin/zsh usuario       # cambiar shell
sudo usermod -l nuevo_nombre viejo_nombre   # renombrar usuario
sudo usermod -L usuario                  # bloquear cuenta (lock)
sudo usermod -U usuario                    # desbloquear cuenta

sudo userdel usuario                 # borrar usuario, mantiene su home
sudo userdel -r usuario                # borrar usuario + su home y mail
```

{% hint style="warning" %}
&#x20;`usermod -G` sin `-a` **sustituye** todos los grupos secundarios del usuario por los indicados, en vez de añadir. Es un error muy común que deja a un usuario fuera de grupos a los que ya pertenecía.&#x20;
{% endhint %}

### Grupos

```bash
sudo groupadd nombre_grupo          # crear grupo
sudo groupdel nombre_grupo            # borrar grupo
sudo gpasswd -a usuario grupo           # añadir usuario a un grupo (alternativa a usermod -aG)
sudo gpasswd -d usuario grupo             # quitar usuario de un grupo

groups usuario                   # ver a qué grupos pertenece
id usuario                         # UID, GID y todos los grupos, de una vez
cat /etc/group                       # listado completo de grupos y miembros
```

### sudo y /etc/sudoers

```bash
sudo -l                    # ver qué puede ejecutar como sudo el usuario actual
sudo -u otro_usuario comando    # ejecutar un comando como otro usuario distinto de root
sudo -i                       # abrir una shell de login como root
sudo -k                         # olvidar la contraseña cacheada (fuerza a pedirla de nuevo)
```

Editar la configuración de sudo **siempre** con `visudo`, nunca editando `/etc/sudoers` directamente:

```bash
sudo visudo
```

```
# Formato básico de una línea de sudoers
usuario    ALL=(ALL:ALL) ALL              # acceso total a sudo
usuario    ALL=(ALL) NOPASSWD: ALL          # sudo sin pedir contraseña
usuario    ALL=(ALL) /usr/bin/systemctl       # solo puede usar sudo con ese comando concreto
%grupo     ALL=(ALL) ALL                        # regla aplicada a todo un grupo (con %)
```

{% hint style="warning" %}
&#x20;`visudo` valida la sintaxis antes de guardar; editar `/etc/sudoers` directamente con un editor normal puede dejarlo con un error de sintaxis que **bloquea el uso de sudo para todo el sistema**, sin forma fácil de arreglarlo sin acceso root alternativo.&#x20;
{% endhint %}

### Enfoque ofensivo: enumeración rápida de privilegios

```bash
sudo -l                                    # qué puede ejecutar el usuario actual con sudo
cat /etc/passwd | grep -v nologin            # usuarios con shell interactiva real
find / -perm -4000 -type f 2>/dev/null         # binarios SUID (ver página de permisos)
getent group sudo                                # miembros del grupo sudo
```

Si `sudo -l` muestra un binario permitido sin contraseña que aparece en [GTFOBins](https://gtfobins.github.io/) con función `sudo`, suele haber una vía directa de escalada.
