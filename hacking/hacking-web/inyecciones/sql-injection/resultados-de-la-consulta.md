# Resultados de la consulta

En esta sección, aprenderemos cómo controlar los resultados de cualquier consulta.

### Ordenar resultados

Podemos ordenar los resultados de cualquier consulta usando [ORDENAR POR](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html) y especificando la columna por la que ordenar:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins ORDER BY password;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)
```

De forma predeterminada, la clasificación se realiza en orden ascendente, pero también podemos ordenar los resultados por `ASC` o `DESC`:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins ORDER BY password DESC;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)
```

También es posible ordenar por varias columnas, para tener una clasificación secundaria para valores duplicados en una columna:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins ORDER BY password DESC, id ASC;

+----+---------------+-----------------+---------------------+
| id | username      | password        | date_of_joining     |
+----+---------------+-----------------+---------------------+
|  1 | admin         | p@ssw0rd        | 2020-07-02 00:00:00 |
|  2 | administrator | change_password | 2020-07-02 11:30:50 |
|  3 | john          | change_password | 2020-07-02 11:47:16 |
|  4 | tom           | change_password | 2020-07-02 11:50:20 |
+----+---------------+-----------------+---------------------+
4 rows in set (0.00 sec)
```

### LIMITAR resultados

Si nuestra consulta devuelve una gran cantidad de registros, podemos [LÍMITE](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html) los resultados a lo que solo queremos, usando `LIMIT` y el número de registros que queremos:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins LIMIT 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```

Si quisiéramos LIMITAR los resultados con un desplazamiento, podríamos especificar el desplazamiento antes del recuento LÍMITE:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins LIMIT 1, 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```

{% hint style="info" %}
Nota: el desplazamiento marca el orden del primer registro a incluir, comenzando desde 0. En el ejemplo anterior, inicia e incluye el segundo registro y devuelve dos valores.
{% endhint %}

### Cláusula WHERE

Para filtrar o buscar datos específicos, podemos utilizar condiciones con la declaración `SELECT` utilizando la cláusula [DONDE](https://dev.mysql.com/doc/refman/8.0/en/where-optimization.html), para afinar los resultados:

```sql
SELECT * FROM table_name WHERE <condition>;
```

La consulta anterior devolverá todos los registros que satisfagan la condición dada. Veamos un ejemplo:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins WHERE id > 1;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
3 rows in set (0.00 sec)
```

El ejemplo anterior selecciona todos los registros donde el valor de `id` es mayor que `1`. Como podemos ver, la primera fila con su `id` 1 fue omitida de la salida. Podemos hacer algo similar para los nombres de usuario:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins where username = 'admin';

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  1 | admin    | p@ssw0rd | 2020-07-02 00:00:00 |
+----+----------+----------+---------------------+
1 row in set (0.00 sec)
```

La consulta anterior selecciona el registro donde está el nombre de usuario `admin`. Podemos utilizar la declaración `UPDATE` para actualizar ciertos registros que cumplen una condición específica.

{% hint style="warning" %}
Nota: Los tipos de datos de cadena y fecha deben estar rodeados por comillas simples (') o comillas dobles ("), mientras que los números se pueden usar directamente.
{% endhint %}

### Cláusula LIKE

Otra cláusula SQL útil es [COMO](https://dev.mysql.com/doc/refman/8.0/en/pattern-matching.html), lo que permite seleccionar registros haciendo coincidir un patrón determinado. La siguiente consulta recupera todos los registros con nombres de usuario que comienzan con `admin`:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins WHERE username LIKE 'admin%';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | administrator | adm1n_p@ss | 2020-07-02 15:19:02 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```

El `%` símbolo actúa como un comodín y coincide con todos los caracteres posteriores a `admin`. Se utiliza para hacer coincidir cero o más caracteres. De manera similar, el `_` símbolo se utiliza para que coincida exactamente con un carácter. La siguiente consulta hace coincidir todos los nombres de usuario con exactamente tres caracteres, lo que en este caso fue `tom`:

Resultados de la consulta

```shell-session
mysql> SELECT * FROM logins WHERE username like '___';

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  3 | tom      | tom123!  | 2020-07-02 15:18:56 |
+----+----------+----------+---------------------+
1 row in set (0.01 sec)
```
