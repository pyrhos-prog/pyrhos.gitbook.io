# Declaraciones SQL

Ahora que entendemos cómo utilizar la utilidad `mysql` y la creación de bases de datos y tablas, veamos algunas de las declaraciones SQL esenciales y sus usos.

### INSERTAR Declaración

El [INSERT](https://dev.mysql.com/doc/refman/8.0/en/insert.html) La declaración se utiliza para agregar nuevos registros a una tabla determinada. La declaración sigue la siguiente sintaxis:

{% code title="Sintaxis (SQL)" %}
```sql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
```
{% endcode %}

La sintaxis anterior requiere que el usuario complete valores para todas las columnas presentes en la tabla.

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');

Query OK, 1 row affected (0.00 sec)
```
{% endcode %}

El ejemplo anterior muestra cómo agregar un nuevo inicio de sesión a la tabla `logins`, con valores apropiados para cada columna. Podemos omitir el llenado de columnas con valores predeterminados (por ejemplo `id` y `date_of_joining`) especificando los nombres de las columnas para insertar valores de forma selectiva:

{% code title="Sintaxis selectiva (SQL)" %}
```sql
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
```
{% endcode %}

{% hint style="info" %}
Omitir columnas con la restricción `NOT NULL` generará un error, ya que es un valor requerido.
{% endhint %}

Podemos hacer lo mismo para insertar valores en la tabla `logins`:

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');

Query OK, 1 row affected (0.00 sec)
```
{% endcode %}

Insertamos un par de nombre de usuario y contraseña en el ejemplo anterior mientras omitimos las columnas `id` y `date_of_joining`.

{% hint style="warning" %}
Los ejemplos insertan contraseñas en texto sin cifrar en la tabla, solo para demostración. Esta es una mala práctica: las contraseñas siempre deben codificarse antes del almacenamiento.
{% endhint %}

También podemos insertar varios registros a la vez separándolos con una coma:

{% code title="Ejemplo múltiples (mysql)" %}
```shell-session
mysql> INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');

Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0
```
{% endcode %}

La consulta anterior insertó dos registros nuevos a la vez.

***

### SELECCIONAR Declaración

Ahora que hemos insertado datos en tablas, veamos cómo recuperar datos con la declaración [SELECT](https://dev.mysql.com/doc/refman/8.0/en/select.html). La sintaxis general para ver la tabla completa es la siguiente:

{% code title="Sintaxis (SQL)" %}
```sql
SELECT * FROM table_name;
```
{% endcode %}

El símbolo de asterisco (\*) actúa como comodín y selecciona todas las columnas. La palabra clave `FROM` indica la tabla desde la que se seleccionará. También es posible ver datos presentes en columnas específicas:

{% code title="Sintaxis columnas específicas (SQL)" %}
```sql
SELECT column1, column2 FROM table_name;
```
{% endcode %}

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> SELECT * FROM logins;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)


mysql> SELECT username,password FROM logins;

+---------------+------------+
| username      | password   |
+---------------+------------+
| admin         | p@ssw0rd   |
| administrator | adm1n_p@ss |
| john          | john123!   |
| tom           | tom123!    |
+---------------+------------+
4 rows in set (0.00 sec)
```
{% endcode %}

La primera consulta muestra todos los registros presentes en la tabla `logins`. La segunda consulta selecciona solo las columnas `username` y `password`, omitiendo las demás.

***

### Declaración DROP

Podemos usar [DROP](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html) para eliminar tablas y bases de datos del servidor.

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> DROP TABLE logins;

Query OK, 0 rows affected (0.01 sec)


mysql> SHOW TABLES;

Empty set (0.00 sec)
```
{% endcode %}

Como podemos ver, la tabla fue eliminada por completo.

{% hint style="danger" %}
La declaración `DROP` eliminará la tabla de forma permanente y completa sin confirmación, por lo que debe usarse con precaución.
{% endhint %}

***

### Declaración ALTER

Podemos utilizar [ALTER](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) para cambiar el nombre de una tabla, cambiar campos, o eliminar o agregar columnas a una tabla existente.

Agregar una nueva columna `newColumn` a la tabla `logins` usando `ADD`:

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> ALTER TABLE logins ADD newColumn INT;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

Para cambiar el nombre de una columna, podemos utilizar `RENAME COLUMN`:

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

También podemos cambiar el tipo de datos de una columna con `MODIFY`:

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> ALTER TABLE logins MODIFY newerColumn DATE;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

Finalmente, podemos eliminar una columna usando `DROP`:

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> ALTER TABLE logins DROP newerColumn;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

Podemos utilizar cualquiera de las declaraciones anteriores con cualquier tabla existente, siempre que tengamos suficientes privilegios para hacerlo.

***

### ACTUALIZAR Declaración

Mientras `ALTER` se utiliza para cambiar las propiedades de una tabla, la declaración [UPDATE](https://dev.mysql.com/doc/refman/8.0/en/update.html) se puede utilizar para actualizar registros específicos dentro de una tabla, según ciertas condiciones. Su sintaxis general es:

{% code title="Sintaxis (SQL)" %}
```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
```
{% endcode %}

Especificamos el nombre de la tabla, cada columna y su nuevo valor, y la condición para actualizar los registros. Veamos un ejemplo:

{% code title="Ejemplo (mysql)" %}
```shell-session
mysql> UPDATE logins SET password = 'change_password' WHERE id > 1;

Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0


mysql> SELECT * FROM logins;

+----+---------------+-----------------+---------------------+
| id | username      | password        | date_of_joining     |
+----+---------------+-----------------+---------------------+
|  1 | admin         | p@ssw0rd        | 2020-07-02 00:00:00 |
|  2 | administrator | change_password | 2020-07-02 11:30:50 |
|  3 | john          | change_password | 2020-07-02 11:47:16 |
|  4 | tom           | change_password | 2020-07-02 11:47:16 |
+----+---------------+-----------------+---------------------+
4 rows in set (0.00 sec)
```
{% endcode %}

La consulta anterior actualizó todas las contraseñas en los registros donde `id > 1`.

{% hint style="info" %}
Al usar `UPDATE`, especificar la cláusula `WHERE` es esencial para determinar qué registros se actualizan.
{% endhint %}
