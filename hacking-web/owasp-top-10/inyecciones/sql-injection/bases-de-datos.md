---
icon: database
---

# Bases de datos

Antes de aprender sobre las inyecciones SQL, necesitamos aprender sobre las bases de datos y el Lenguaje SQL. Las aplicaciones web utilizan bases de datos backend para almacenar contenido e información relacionados con ellas:

* recursos esenciales de la web
* imágenes y archivos contenido como
* publicaciones y actualizaciones
* datos de usuario (nombres de usuario y contraseñas)

Existen diferentes bases de datos, cada una adaptada a un uso específico. Antes las aplicaciones utilizaban bases de datos basadas en archivos, lo cual resultaba muy lento con el aumento de tamaño. Esto condujo a la adopción de `Database Management Systems` (`DBMS`).

### Sistemas de gestión de bases de datos

Un Sistema de Gestión de Bases de Datos (SGBD) ayuda a crear, definir, alojar y gestionar bases de datos. A lo largo del tiempo se han diseñado diversos tipos de SGBD:

* basados ​​en archivos
* SGBD relacionales (SGBDR)
* NoSQL
* basados ​​en grafos
* almacenes de clave-valor.

Existen múltiples maneras de interactuar con un SGBD, como herramientas de línea de comandos, interfaces gráficas o incluso API (Interfaces de Programación de Aplicaciones).

Algunas de las características esenciales de un SGBD incluyen:

| **Característica** | **Descripción**                                                                                                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Concurrencia`     | Una aplicación real puede tener varios usuarios interactuando simultáneamente. Un SGBD garantiza que estas interacciones concurrentes se realicen correctamente sin dañar ni perder datos. |
| `Consistencia`     | El DBMS debe garantizar que los datos permanezcan consistentes y válidos en toda la base de datos.                                                                                         |
| `Security`         | El SGBD proporciona controles de seguridad detallados mediante la autenticación y los permisos de usuario.                                                                                 |
| `Reliability`      | Es fácil realizar copias de seguridad de bases de datos y revertirlas a un estado anterior.                                                                                                |
| `Lenguaje SQL`     | Sintaxis intuitiva que admite diversas operaciones.                                                                                                                                        |

### Arquitectura

El diagrama a continuación detalla una arquitectura de dos niveles.

![Diagrama de arquitectura de tres niveles: El usuario interactúa con una aplicación cliente en el Nivel I, que se conecta a un servidor de aplicaciones en el Nivel II y a un SGBD en el Nivel III. Los usuarios y un administrador de base de datos acceden al SGBD.](https://content.gitbook.com/content/6EVQrWLD8melKOyrWHcr/blobs/sN1OnDiwQCU4oc5FNwuc/db_2.png)

`Tier I` generalmente consiste en aplicaciones del lado del cliente, como sitios web o programas con interfaz gráfica de usuario (GUI). Estas aplicaciones consisten en interacciones de alto nivel, como el inicio de sesión o los comentarios de los usuarios. Los datos de estas interacciones se transmiten a `Tier II` mediante llamadas a la API u otras solicitudes.
