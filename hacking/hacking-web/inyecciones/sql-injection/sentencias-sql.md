# Sentencias SQL

Ahora que entendemos cómo utilizar la `mysql` utilidad y crear bases de datos y tablas, veamos algunas de las declaraciones SQL esenciales y sus usos.

### Declaración INSERT

La instrucción [INSERT](https://dev.mysql.com/doc/refman/8.0/en/insert.html) se utiliza para agregar nuevos registros a una tabla. La instrucción sigue la siguiente sintaxis:

{% code title="Sintaxis" %}
```sql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
```
{% endcode %}

La sintaxis anterior requiere que el usuario complete los valores para todas las columnas presentes en la tabla.

{% code title="Ejemplo" %}
```shell-session
mysql> INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');

Query OK, 1 row affected (0.00 sec)
```
{% endcode %}

El ejemplo anterior muestra cómo agregar un nuevo inicio de sesión a la tabla de inicios de sesión, con los valores adecuados para cada columna. Sin embargo, podemos omitir el llenado de las columnas con valores predeterminados, como `id` y `date_of_joining`. Esto se puede hacer especificando los nombres de las columnas para insertar valores en una tabla de forma selectiva:

{% code title="Sintaxis (columnas específicas)" %}
```sql
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
```
{% endcode %}

{% hint style="info" %}
Omitir columnas con la restricción NOT NULL generará un error, ya que es un valor obligatorio.
{% endhint %}

Podemos hacer lo mismo para insertar valores en la `logins` tabla:

{% code title="Ejemplo" %}
```shell-session
mysql> INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');

Query OK, 1 row affected (0.00 sec)
```
{% endcode %}

Insertamos un par de nombre de usuario-contraseña en el ejemplo anterior, omitiendo las columnas `id` y `date_of_joining`.

{% hint style="warning" %}
Los ejemplos insertan contraseñas de texto sin cifrar en la tabla, solo a modo de demostración. Esto es una mala práctica, ya que las contraseñas siempre deben cifrarse antes de almacenarse.
{% endhint %}

También podemos insertar varios registros a la vez separándolos con una coma:

{% code title="Ejemplo (múltiples registros)" %}
```shell-session
mysql> INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');

Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0
```
{% endcode %}

La consulta anterior insertó dos nuevos registros a la vez.

***

### Sentencia SELECT

Ahora que hemos insertado datos en las tablas, veamos cómo recuperarlos con la instrucción [SELECT](https://dev.mysql.com/doc/refman/8.0/en/select.html). Esta instrucción también puede utilizarse para muchos otros fines, que veremos más adelante. La sintaxis general para ver la tabla completa es la siguiente:

{% code title="Sintaxis (todas las columnas)" %}
```sql
SELECT * FROM table_name;
```
{% endcode %}

El asterisco (\*) actúa como comodín y selecciona todas las columnas. La `FROM` palabra clave se utiliza para indicar la tabla de la que se selecciona. También es posible ver los datos presentes en columnas específicas:

{% code title="Sintaxis (columnas específicas)" %}
```sql
SELECT column1, column2 FROM table_name;
```
{% endcode %}

La consulta anterior seleccionará los datos presentes únicamente en las columnas 1 y 2.

{% code title="Ejemplo" %}
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

La primera consulta del ejemplo anterior examina todos los registros presentes en la tabla de inicios de sesión. Podemos ver los cuatro registros ingresados ​​anteriormente. La segunda consulta selecciona solo las columnas de nombre de usuario y contraseña, omitiendo las otras dos.

***

### Declaración DROP

Podemos usar [DROP](https://dev.mysql.com/doc/refman/8.0/en/drop-table.html) para eliminar tablas y bases de datos del servidor.

{% code title="Ejemplo" %}
```shell-session
mysql> DROP TABLE logins;

Query OK, 0 rows affected (0.01 sec)


mysql> SHOW TABLES;

Empty set (0.00 sec)
```
{% endcode %}

Como podemos observar la tabla fue eliminada por completo.

La declaración 'DROP' eliminará la tabla de forma permanente y completa sin confirmación, por lo que debe utilizarse con precaución.

***

### Declaración ALTER

Finalmente, podemos usar [ALTER](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) para cambiar el nombre de cualquier tabla y cualquiera de sus campos, o para eliminar o añadir una nueva columna a una tabla existente. El siguiente ejemplo añade una nueva columna `newColumn` a la `logins` tabla usando `ADD`:

{% code title="Ejemplo (ADD)" %}
```shell-session
mysql> ALTER TABLE logins ADD newColumn INT;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

Para cambiar el nombre de una columna, podemos utilizar `RENAME COLUMN`:

{% code title="Ejemplo (RENAME COLUMN)" %}
```shell-session
mysql> ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

También podemos cambiar el tipo de datos de una columna con `MODIFY`:

{% code title="Ejemplo (MODIFY)" %}
```shell-session
mysql> ALTER TABLE logins MODIFY newerColumn DATE;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

Finalmente, podemos eliminar una columna usando `DROP`:

{% code title="Ejemplo (DROP column)" %}
```shell-session
mysql> ALTER TABLE logins DROP newerColumn;

Query OK, 0 rows affected (0.01 sec)
```
{% endcode %}

Podemos utilizar cualquiera de las declaraciones anteriores con cualquier tabla existente, siempre que tengamos suficientes privilegios para hacerlo.

***

### Declaración UPDATE

Aunque `ALTER` se utiliza para cambiar las propiedades de una tabla, la instrucción [UPDATE](https://dev.mysql.com/doc/refman/8.0/en/update.html) puede utilizarse para actualizar registros específicos dentro de una tabla, según ciertas condiciones. Su sintaxis general es:

{% code title="Sintaxis" %}
```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
```
{% endcode %}

Especificamos el nombre de la tabla, cada columna y su nuevo valor, y la condición para actualizar los registros. Veamos un ejemplo:

{% code title="Ejemplo" %}
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

La consulta anterior actualizó todas las contraseñas en todos los registros donde el id era mayor que 1.

{% hint style="info" %}
Debemos especificar la cláusula WHERE con UPDATE para indicar qué registros se actualizan.
{% endhint %}
