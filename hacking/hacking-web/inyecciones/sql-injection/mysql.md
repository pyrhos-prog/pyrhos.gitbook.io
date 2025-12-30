# MYSQL

Este módulo presenta la inyección SQL mediante \[nombre del módulo] `MySQL`. Es fundamental aprender más sobre `MySQL`/SQL para comprender cómo funcionan y utilizarlas correctamente. Por lo tanto, esta sección cubrirá algunos conceptos básicos y la sintaxis de MySQL/SQL, así como ejemplos de su uso en bases de datos MySQL/MariaDB.

***

### Lenguaje de consulta estructurado (SQL)

La sintaxis SQL puede variar de un RDBMS a otro. Sin embargo, todos deben seguir el [estándar ISO](https://en.wikipedia.org/wiki/ISO/IEC_9075) para Lenguaje de Consulta Estructurado. En los ejemplos mostrados, seguiremos la sintaxis de MySQL/MariaDB. SQL permite realizar las siguientes acciones:

* Recuperar datos
* Actualizar datos
* Eliminar datos
* Crear nuevas tablas y bases de datos
* Agregar/eliminar usuarios
* Asignar permisos a estos usuarios

***

### Línea de comandos

La utilidad `mysql` se utiliza para autenticarse e interactuar con una base de datos MySQL/MariaDB. El indicador `-u` se utiliza para proporcionar el nombre de usuario y el `-p` para la contraseña. El `-p` debe pasarse vacío (sin contraseña en el propio comando), para que se nos solicite introducir la contraseña y no quede almacenada en texto plano en el historial de la terminal.

{% code title="Inicio de sesión interactivo" %}
```shell
OsmanMartinez@htb[/htb]$ mysql -u root -p

Enter password: <password>
...SNIP...

mysql> 
```
{% endcode %}

También es posible usar la contraseña directamente en el comando (no recomendado porque puede quedar registrada en el historial):

{% code title="Inicio de sesión con contraseña en línea (no recomendado)" %}
```shell
OsmanMartinez@htb[/htb]$ mysql -u root -p<password>

...SNIP...

mysql> 
```
{% endcode %}

{% hint style="info" %}
Consejo: No debe haber espacios entre `-p` y la contraseña.
{% endhint %}

Los ejemplos anteriores permiten iniciar sesión como superusuario, es decir, `root`, con la contraseña `password`, para tener privilegios para ejecutar todos los comandos. Otros usuarios de DBMS tendrían ciertos privilegios sobre las sentencias que pueden ejecutar. Podemos ver nuestros privilegios usando el comando [SHOW GRANTS](https://dev.mysql.com/doc/refman/8.0/en/show-grants.html), que se explicará más adelante.

Si no especificamos un host, se usará el `localhost` por defecto. Podemos especificar un host y un puerto remotos mediante los indicadores `-h` y `-P`.

{% code title="Conexión remota (host y puerto)" %}
```shell
OsmanMartinez@htb[/htb]$ mysql -u root -h docker.hackthebox.eu -P 3306 -p 

Enter password: 
...SNIP...

mysql> 
```
{% endcode %}

{% hint style="info" %}
Nota: El puerto predeterminado de MySQL/MariaDB es 3306. Se especifica con una `P` mayúscula (`-P`), distinta de la `p` minúscula usada para la contraseña (`-p`).
{% endhint %}

{% hint style="warning" %}
Nota: Para seguir los ejemplos, intenta usar la herramienta `mysql` en tu PwnBox para iniciar sesión en el SGBD que se encuentra en la pregunta al final de la sección, usando su IP y puerto. Usa `root` como nombre de usuario y `password` como contraseña.
{% endhint %}

***

### Creando una base de datos

Una vez que iniciamos sesión en la base de datos con la utilidad `mysql`, podemos empezar a usar consultas SQL para interactuar con el SGBD. Por ejemplo, se puede crear una nueva base de datos dentro del SGBD MySQL mediante la instrucción [CREATE DATABASE](https://dev.mysql.com/doc/refman/5.7/en/create-database.html).

{% code title="CREATE DATABASE" %}
```shell
mysql> CREATE DATABASE users;

Query OK, 1 row affected (0.02 sec)
```
{% endcode %}

MySQL espera que las consultas de línea de comandos terminen con un punto y coma. El ejemplo anterior creó una nueva base de datos llamada `users`. Podemos ver la lista de bases de datos con [SHOW DATABASES](https://dev.mysql.com/doc/refman/8.0/en/show-databases.html) y acceder a la `users` base de datos con la instrucción `USE`:

{% code title="SHOW DATABASES y USE" %}
```shell
mysql> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |
+--------------------+

mysql> USE users;

Database changed
```
{% endcode %}

Las sentencias SQL no distinguen entre mayúsculas y minúsculas, lo que significa que `USE users;` y `use users;` se refieren al mismo comando. Sin embargo, el nombre de la base de datos sí distingue entre mayúsculas y minúsculas, por lo que no podemos usar `USE USERS;` en lugar de `USE users;`. Por lo tanto, es recomendable especificar las sentencias en mayúsculas para evitar confusiones.

***

### Tablas

Un SGBD almacena datos en forma de tablas. Una tabla se compone de filas horizontales y columnas verticales. La intersección de una fila y una columna se denomina celda. Cada tabla se crea con un conjunto fijo de columnas, cada una de las cuales corresponde a un tipo de dato específico.

Un tipo de dato define el tipo de valor que debe contener una columna. Ejemplos comunes son `numbers`, `strings`, `date`, `time` y `binary data`. También puede haber tipos de datos específicos de DBMS. Puede encontrar una lista completa de tipos de datos en MySQL [aquí](https://dev.mysql.com/doc/refman/8.0/en/data-types.html).

Por ejemplo, creemos una tabla `logins` para almacenar datos de usuario mediante la consulta SQL [CREATE TABLE](https://dev.mysql.com/doc/refman/8.0/en/creating-tables.html):

{% code title="CREATE TABLE logins" %}
```sql
CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME
    );
```
{% endcode %}

Como podemos ver, la `CREATE TABLE` consulta primero especifica el nombre de la tabla y luego (entre paréntesis) especificamos cada columna por su nombre y tipo de dato, separados por comas. Después del nombre y el tipo, podemos especificar propiedades específicas, como se explicará más adelante.

{% code title="Resultado CREATE TABLE" %}
```shell
mysql> CREATE TABLE logins (
    ->     id INT,
    ->     username VARCHAR(100),
    ->     password VARCHAR(100),
    ->     date_of_joining DATETIME
    ->     );
Query OK, 0 rows affected (0.03 sec)
```
{% endcode %}

Las consultas anteriores crean una tabla `logins` con cuatro columnas. La primera columna `id` es un entero. Las dos columnas siguientes `username` y `password` son cadenas de 100 caracteres cada una. Cualquier entrada con una longitud mayor generará un error. La columna `date_of_joining` de tipo `DATETIME` almacena la fecha en que se agregó una entrada.

{% code title="SHOW TABLES" %}
```shell
mysql> SHOW TABLES;

+-----------------+
| Tables_in_users |
+-----------------+
| logins          |
+-----------------+
1 row in set (0.00 sec)
```
{% endcode %}

Se puede obtener una lista de las tablas de la base de datos actual mediante la instrucción `SHOW TABLES`. Además, la palabra clave [DESCRIBE](https://dev.mysql.com/doc/refman/8.0/en/describe.html) se utiliza para listar la estructura de la tabla con sus campos y tipos de datos.

{% code title="DESCRIBE logins" %}
```shell
mysql> DESCRIBE logins;

+-----------------+--------------+
| Field           | Type         |
+-----------------+--------------+
| id              | int          |
| username        | varchar(100) |
| password        | varchar(100) |
| date_of_joining | date         |
+-----------------+--------------+
4 rows in set (0.00 sec)
```
{% endcode %}

#### Propiedades de la tabla

Dentro de la `CREATE TABLE` consulta, se pueden configurar numerosas [propiedades](https://dev.mysql.com/doc/refman/8.0/en/create-table.html) para la tabla y cada columna. Por ejemplo, podemos configurar la `id` columna para que se incremente automáticamente mediante la palabra clave `AUTO_INCREMENT`, lo que incrementa automáticamente el ID en uno cada vez que se añade un nuevo elemento a la tabla:

{% code title="AUTO_INCREMENT y NOT NULL" %}
```sql
    id INT NOT NULL AUTO_INCREMENT,
```
{% endcode %}

La restricción `NOT NULL` garantiza que una columna específica nunca quede vacía (es decir, un campo obligatorio). También podemos usar `UNIQUE` para garantizar que el elemento insertado sea siempre único. Por ejemplo, si la usamos con la `username` columna, podemos asegurar que dos usuarios no tengan el mismo nombre de usuario:

{% code title="UNIQUE y NOT NULL en username" %}
```sql
    username VARCHAR(100) UNIQUE NOT NULL,
```
{% endcode %}

Otra palabra clave importante es `DEFAULT`, que se utiliza para especificar el valor predeterminado. Por ejemplo, dentro de la `date_of_joining` columna, podemos establecer el valor predeterminado en `NOW()`, que en MySQL devuelve la fecha y hora actuales:

{% code title="DEFAULT NOW()" %}
```sql
    date_of_joining DATETIME DEFAULT NOW(),
```
{% endcode %}

Finalmente, una de las propiedades más importantes es `PRIMARY KEY`, que permite identificar de forma única cada registro de la tabla. Podemos convertir la `id` columna en `PRIMARY KEY` para esta tabla:

{% code title="PRIMARY KEY (id)" %}
```sql
    PRIMARY KEY (id)
```
{% endcode %}

La consulta final `CREATE TABLE` será la siguiente:

{% code title="CREATE TABLE final" %}
```sql
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );
```
{% endcode %}

{% hint style="info" %}
Nota: Espere entre 10 y 15 segundos para que los servidores en las preguntas se inicien, para dar tiempo suficiente a que Apache/MySQL se inicie y se ejecute.
{% endhint %}
