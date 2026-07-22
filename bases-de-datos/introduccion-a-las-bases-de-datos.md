---
icon: database
---

# Introducción a las bases de datos

> Una base de datos es una colección organizada de datos, almacenada y accesible electrónicamente, diseñada para que se pueda consultar, insertar, actualizar y eliminar información de forma eficiente y consistente. Frente a guardar datos en archivos planos (CSV, TXT), una base de datos gestionada por un motor (DBMS) aporta:

| Ventaja                  | Por qué importa                                                                 |
| ------------------------ | ------------------------------------------------------------------------------- |
| Integridad               | Restricciones (claves, tipos de dato, unicidad) evitan datos inconsistentes     |
| Concurrencia             | Múltiples usuarios/procesos pueden leer y escribir a la vez sin corromper datos |
| Recuperación ante fallos | Transacciones y logs permiten deshacer cambios o recuperar tras un corte        |
| Rendimiento a escala     | Índices y motores de consulta optimizados para grandes volúmenes                |
| Seguridad granular       | Permisos por usuario, rol, tabla o incluso columna                              |

### Modelo relacional y no relacional

|                 | Relacional (SQL)                                                            | No relacional (NoSQL)                                                          |
| --------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Estructura      | Tablas con filas y columnas, esquema fijo                                   | Documentos, clave-valor, grafos o columnas anchas; esquema flexible            |
| Relaciones      | Explícitas, mediante claves foráneas                                        | Normalmente embebidas o gestionadas a nivel de aplicación                      |
| Consistencia    | Fuerte (ACID)                                                               | Habitualmente eventual (modelo BASE), varía según motor                        |
| Escalado típico | Vertical (más recursos en un servidor)                                      | Horizontal (más nodos)                                                         |
| Ejemplos        | MySQL, PostgreSQL, MSSQL, Oracle, SQLite                                    | MongoDB, Redis, Cassandra, Neo4j                                               |
| Cuándo usarlo   | Datos estructurados con relaciones claras, necesidad de integridad estricta | Datos semiestructurados, alta escritura/lectura distribuida, esquema cambiante |

### Conceptos básicos del modelo relacional

* **Tabla**: conjunto de datos organizados en filas y columnas, equivalente a una entidad (ej. `usuarios`, `pedidos`).
* **Fila (registro/tupla)**: una instancia concreta de esa entidad (ej. un usuario concreto).
* **Columna (campo/atributo)**: una propiedad de la entidad (ej. `nombre`, `email`), con un tipo de dato definido.
* **Clave primaria (Primary Key, PK)**: columna (o combinación) que identifica de forma única cada fila. No admite nulos ni duplicados.
* **Clave foránea (Foreign Key, FK)**: columna que referencia la PK de otra tabla, estableciendo una relación entre ambas.
* **Relaciones**: uno-a-uno, uno-a-muchos, muchos-a-muchos (esta última requiere una tabla intermedia).
* **Normalización**: proceso de organizar las tablas para reducir redundancia y dependencias inconsistentes, dividido en formas normales (1FN, 2FN, 3FN...). A mayor normalización, menos duplicidad pero más JOINs necesarios.
* **Índice**: estructura auxiliar que acelera búsquedas sobre una o varias columnas, a costa de espacio y de ralentizar ligeramente escrituras.
* **Transacción**: conjunto de operaciones que se ejecutan como una unidad atómica — o se aplican todas, o ninguna (propiedades ACID).

#### Propiedades ACID

| Propiedad    | Significado                                                                         |
| ------------ | ----------------------------------------------------------------------------------- |
| Atomicidad   | Una transacción se ejecuta completa o no se ejecuta en absoluto                     |
| Consistencia | La base de datos pasa de un estado válido a otro estado válido                      |
| Aislamiento  | Transacciones concurrentes no interfieren entre sí                                  |
| Durabilidad  | Una vez confirmada (commit), la transacción persiste aunque haya un fallo posterior |

### Sistemas gestores de bases de datos (SGBD/DBMS)

Un DBMS es el software que gestiona el almacenamiento, acceso y seguridad de las bases de datos (MySQL, PostgreSQL, MSSQL...). Se encarga de traducir las consultas SQL en operaciones sobre el almacenamiento físico, gestionar la concurrencia, aplicar los permisos y mantener la integridad definida en el esquema.

