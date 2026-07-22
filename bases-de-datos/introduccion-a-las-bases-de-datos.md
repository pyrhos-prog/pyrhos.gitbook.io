# Introducción a las bases de datos

Una base de datos es un sistema organizado para almacenar, gestionar y recuperar datos de forma persistente.

| Tipo                  | Ejemplos                                 | Lenguaje de consulta     |
| --------------------- | ---------------------------------------- | ------------------------ |
| Relacional (SQL)      | MySQL, PostgreSQL, MSSQL, Oracle, SQLite | SQL                      |
| No relacional (NoSQL) | MongoDB, Redis, Cassandra, Elasticsearch | Específico de cada motor |

### Modelo relacional

* **Tabla**: conjunto de filas y columnas que representa una entidad (ej. `usuarios`).
* **Fila (registro)**: una instancia concreta de esa entidad.
* **Columna (campo)**: un atributo de la entidad (ej. `username`, `password_hash`).
* **Clave primaria (PK)**: identificador único de cada fila.
* **Clave foránea (FK)**: referencia a la PK de otra tabla, define relaciones entre tablas.

```sql
CREATE TABLE usuarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    rol_id INT,
    FOREIGN KEY (rol_id) REFERENCES roles(id)
);
```

### SQL básico

#### Consultas (SELECT)

```sql
SELECT username, rol_id FROM usuarios;
SELECT * FROM usuarios WHERE id = 1;
SELECT * FROM usuarios ORDER BY username DESC LIMIT 10;
```

#### Filtros y operadores comunes

```sql
SELECT * FROM usuarios WHERE username = 'admin' AND rol_id = 1;
SELECT * FROM usuarios WHERE username LIKE '%admin%';
SELECT * FROM usuarios WHERE id IN (1, 2, 3);
```

#### Joins (relación entre tablas)

```sql
SELECT u.username, r.nombre AS rol
FROM usuarios u
JOIN roles r ON u.rol_id = r.id;
```

| Tipo de JOIN | Descripción                                     |
| ------------ | ----------------------------------------------- |
| `INNER JOIN` | Solo filas con coincidencia en ambas tablas     |
| `LEFT JOIN`  | Todas las filas de la izquierda, coincidan o no |
| `RIGHT JOIN` | Todas las filas de la derecha, coincidan o no   |
| `FULL JOIN`  | Todas las filas de ambas tablas                 |

#### Inserción, actualización y borrado

```sql
INSERT INTO usuarios (username, password_hash, rol_id) VALUES ('bob', '<hash>', 2);
UPDATE usuarios SET rol_id = 1 WHERE id = 5;
DELETE FROM usuarios WHERE id = 5;
```

### Metadatos útiles en auditorías

Tablas de sistema que suelen consultarse durante enumeración o explotación (ej. tras una inyección SQL):

| Motor      | Consulta para listar bases de datos | Consulta para listar tablas                    |
| ---------- | ----------------------------------- | ---------------------------------------------- |
| MySQL      | `SHOW DATABASES;`                   | `SHOW TABLES;`                                 |
| MSSQL      | `SELECT name FROM sys.databases;`   | `SELECT name FROM sysobjects WHERE xtype='U';` |
| PostgreSQL | `SELECT datname FROM pg_database;`  | `SELECT tablename FROM pg_tables;`             |

```sql
-- Enumeración típica vía information_schema (MySQL, MSSQL, PostgreSQL)
SELECT table_name FROM information_schema.tables WHERE table_schema = 'nombre_bd';
SELECT column_name FROM information_schema.columns WHERE table_name = 'usuarios';
```

### Relevancia ofensiva

* Las bases de datos mal configuradas (autenticación por defecto, puertos expuestos) son un vector de acceso directo.
* Entender SQL es prerrequisito para SQL injection: sin dominar `SELECT`, `UNION`, subconsultas y `information_schema`, no se puede construir ni interpretar un payload de inyección.
* Los hashes de contraseñas casi siempre viven en una tabla de usuarios — de ahí la conexión directa con ataques de fuerza bruta y cracking (ver Hydra/Hashcat).

### Próximos pasos

Esta página es la base para el módulo de SQL Injection: identificación de puntos de inyección, técnicas UNION-based, error-based, blind (boolean/time), y uso de sqlmap.

***

**Ver también**: Fuzzing web con FFuF · Ataques de contraseñas (Hydra, Medusa, Hashcat) · Herramientas de proxy web (Burp Suite, Caido, OWASP ZAP)
