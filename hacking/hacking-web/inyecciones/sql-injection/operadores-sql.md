# Operadores SQL

A veces, las expresiones con una sola condición no son suficientes para satisfacer los requisitos del usuario. Para ello, SQL admite [Operadores lógicos](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html) para utilizar múltiples condiciones a la vez. Los operadores lógicos más comunes son `AND`, `OR`, y `NOT`.

### Y Operador

El `AND` operador acepta dos condiciones y regresa `true` o `false` basado en su evaluación:

```sql
condition1 AND condition2
```

El resultado de la `AND` operación es `true` si y sólo si ambos `condition1` y `condition2` evalúan a `true`:

{% code title="Operadores SQL" %}
```shell-session
mysql> SELECT 1 = 1 AND 'test' = 'test';

+---------------------------+
| 1 = 1 AND 'test' = 'test' |
+---------------------------+
|                         1 |
+---------------------------+
1 row in set (0.00 sec)

mysql> SELECT 1 = 1 AND 'test' = 'abc';

+--------------------------+
| 1 = 1 AND 'test' = 'abc' |
+--------------------------+
|                        0 |
+--------------------------+
1 row in set (0.00 sec)
```
{% endcode %}

En términos de MySQL, cualquier `non-zero` se considera el valor `true`, y normalmente devuelve el valor `1` para significar `true`. `0` se considera `false`. Como podemos ver en el ejemplo anterior, la primera consulta devolvió `true` ya que ambas expresiones fueron evaluadas como `true`. Sin embargo, la segunda consulta regresó `false` ya que la segunda condición `'test' = 'abc'` es `false`.

### Operador OR

El `OR` operador también toma dos expresiones y devuelve `true` cuando al menos una de ellas evalúa `true`:

{% code title="Operadores SQL" %}
```shell-session
mysql> SELECT 1 = 1 OR 'test' = 'abc';

+-------------------------+
| 1 = 1 OR 'test' = 'abc' |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set (0.00 sec)

mysql> SELECT 1 = 2 OR 'test' = 'abc';

+-------------------------+
| 1 = 2 OR 'test' = 'abc' |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)
```
{% endcode %}

Las consultas anteriores demuestran cómo trabaja el `OR`. La primera consulta evaluó a `true` porque la condición `1 = 1` es `true`. La segunda consulta tiene dos condiciones `false`, dando como resultado `false`.

### NO Operador

El `NOT` operador simplemente invierte un valor booleano — es decir `true` se convierte en `false` y viceversa:

{% code title="Operadores SQL" %}
```shell-session
mysql> SELECT NOT 1 = 1;

+-----------+
| NOT 1 = 1 |
+-----------+
|         0 |
+-----------+
1 row in set (0.00 sec)

mysql> SELECT NOT 1 = 2;

+-----------+
| NOT 1 = 2 |
+-----------+
|         1 |
+-----------+
1 row in set (0.00 sec)
```
{% endcode %}

Como se ve en los ejemplos anteriores, la primera consulta dio como resultado `false` porque es lo inverso de la evaluación de `1 = 1`, que es `true`. Por otro lado, la segunda consulta devolvió `true`, ya que la inversa de `1 = 2` (que es `false`) es `true`.

### Operadores de símbolos

Los operadores `AND`, `OR` y `NOT` también pueden representarse como `&&`, `||` y `!`, respectivamente. Los siguientes son los mismos ejemplos anteriores, utilizando los operadores de símbolos:

{% code title="Operadores SQL" %}
```shell-session
mysql> SELECT 1 = 1 && 'test' = 'abc';

+-------------------------+
| 1 = 1 && 'test' = 'abc' |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SELECT 1 = 1 || 'test' = 'abc';

+-------------------------+
| 1 = 1 || 'test' = 'abc' |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SELECT 1 != 1;

+--------+
| 1 != 1 |
+--------+
|      0 |
+--------+
1 row in set (0.00 sec)
```
{% endcode %}

### Operadores en consultas

Veamos cómo se pueden utilizar estos operadores en las consultas. La siguiente consulta enumera todos los registros donde `username` NO es `john`:

{% code title="Operadores SQL" %}
```shell-session
mysql> SELECT * FROM logins WHERE username != 'john';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
3 rows in set (0.00 sec)
```
{% endcode %}

La siguiente consulta selecciona usuarios que tienen su `id` mayor que `1` Y `username` NO igual a `john`:

{% code title="Operadores SQL" %}
```shell-session
mysql> SELECT * FROM logins WHERE username != 'john' AND id > 1;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```
{% endcode %}

### Precedencia de múltiples operadores

SQL admite varias otras operaciones, como suma, división y operaciones bit a bit. Por tanto, una consulta podría tener múltiples expresiones con múltiples operaciones a la vez. El orden de estas operaciones se decide mediante la precedencia del operador.

Aquí hay una lista de operaciones comunes y su precedencia, como se ve en la [Documentación de MariaDB](https://mariadb.com/kb/en/operator-precedence/):

* División (`/`), Multiplicación (`*`), y Módulo (`%`)
* Adición (`+`) y resta (`-`)
* Comparación (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
* NO (`!`)
* Y (`&&`)
* O (`||`)

Las operaciones en la parte superior se evalúan antes que las de la parte inferior de la lista. Veamos un ejemplo:

```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
```

La consulta tiene cuatro operaciones: `!=`, `AND`, `>`, y `-`. Por la precedencia del operador, sabemos que la resta viene primero, por lo que primero se evaluará `3 - 2` a `1`:

```sql
SELECT * FROM logins WHERE username != 'tom' AND id > 1;
```

A continuación, tenemos dos operaciones de comparación, `>` y `!=`. Ambos tienen la misma precedencia y serán evaluados juntos. Entonces, devolverá todos los registros donde el nombre de usuario no sea `tom`, y todos los registros donde el `id` es mayor que 1 y luego aplicará `AND` para devolver todos los registros que cumplen ambas condiciones:

{% code title="Operadores SQL" %}
```shell-session
mysql> select * from logins where username != 'tom' AND id > 3 - 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-03 12:03:53 |
|  3 | john          | john123!   | 2020-07-03 12:03:57 |
+----+---------------+------------+---------------------+
2 rows in set (0.00 sec)
```
{% endcode %}

Veremos algunos otros escenarios de precedencia de operadores en las próximas secciones.
