---
icon: database
---

# Lenguaje SQL



> SQL (Structured Query Language) es el lenguaje estándar para interactuar con bases de datos relacionales. Se divide en subconjuntos según el tipo de operación que realizan. La sintaxis base es común entre motores, con variaciones puntuales.

### DDL — Data Definition Language

Define y modifica la estructura de la base de datos (esquemas, tablas, índices).

| Comando    | Uso                                        | Ejemplo                                                           |
| ---------- | ------------------------------------------ | ----------------------------------------------------------------- |
| `CREATE`   | Crear una base de datos, tabla, índice...  | `CREATE TABLE usuarios (id INT PRIMARY KEY, nombre VARCHAR(50));` |
| `ALTER`    | Modificar una estructura existente         | `ALTER TABLE usuarios ADD COLUMN email VARCHAR(100);`             |
| `DROP`     | Eliminar una estructura completa           | `DROP TABLE usuarios;`                                            |
| `TRUNCATE` | Vaciar una tabla manteniendo su estructura | `TRUNCATE TABLE logs;`                                            |

### DML — Data Manipulation Language

Manipula los datos contenidos en las tablas.

| Comando  | Uso                        | Ejemplo                                                  |
| -------- | -------------------------- | -------------------------------------------------------- |
| `INSERT` | Añadir filas               | `INSERT INTO usuarios (id, nombre) VALUES (1, 'Osman');` |
| `UPDATE` | Modificar filas existentes | `UPDATE usuarios SET nombre='Pyrhos' WHERE id=1;`        |
| `DELETE` | Eliminar filas             | `DELETE FROM usuarios WHERE id=1;`                       |

`UPDATE` y `DELETE` sin `WHERE` afectan a toda la tabla — error común y peligroso tanto en administración como fuente de bugs explotables.

### DQL — Data Query Language

Consulta los datos. En la práctica es el subconjunto más usado y el más relevante para SQLi.

```sql
SELECT columna1, columna2
FROM tabla
WHERE condicion
GROUP BY columna
HAVING condicion_agregada
ORDER BY columna ASC|DESC
LIMIT n;
```

| Cláusula                                  | Función                                                                  |
| ----------------------------------------- | ------------------------------------------------------------------------ |
| `WHERE`                                   | Filtra filas antes de agrupar                                            |
| `JOIN` (`INNER`, `LEFT`, `RIGHT`, `FULL`) | Combina filas de varias tablas según una condición de relación           |
| `GROUP BY`                                | Agrupa filas para aplicar funciones agregadas (`COUNT`, `SUM`, `AVG`...) |
| `HAVING`                                  | Filtra después de agrupar (equivalente a `WHERE` mas para grupos)        |
| `ORDER BY`                                | Ordena el resultado                                                      |
| `LIMIT` / `OFFSET`                        | Pagina resultados                                                        |
| Subconsultas                              | `SELECT` anidado dentro de otra consulta, en `WHERE`, `FROM` o `SELECT`  |

Ejemplo de `JOIN`:

```sql
SELECT pedidos.id, usuarios.nombre
FROM pedidos
INNER JOIN usuarios ON pedidos.usuario_id = usuarios.id;
```

### DCL — Data Control Language

Controla el acceso y los permisos sobre los objetos de la base de datos.

| Comando  | Uso               | Ejemplo                                                         |
| -------- | ----------------- | --------------------------------------------------------------- |
| `GRANT`  | Conceder permisos | `GRANT SELECT, INSERT ON basedatos.* TO 'usuario'@'localhost';` |
| `REVOKE` | Retirar permisos  | `REVOKE INSERT ON basedatos.* FROM 'usuario'@'localhost';`      |

Relacionado con TCL (Transaction Control Language): `COMMIT` (confirma cambios), `ROLLBACK` (deshace cambios no confirmados), `SAVEPOINT` (punto intermedio dentro de una transacción).
