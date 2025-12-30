# Bases de datos

Antes de aprender sobre las inyecciones SQL, necesitamos aprender más sobre las bases de datos y el Lenguaje de Consulta Estructurado (SQL), que realizará las consultas necesarias. Las aplicaciones web utilizan bases de datos backend para almacenar contenido e información relacionados con ellas. Estos pueden ser recursos esenciales de la aplicación web, como imágenes y archivos; contenido como publicaciones y actualizaciones; o datos de usuario, como nombres de usuario y contraseñas.

Existen muchos tipos diferentes de bases de datos, cada una adaptada a un uso específico. Tradicionalmente, las aplicaciones utilizaban bases de datos basadas en archivos, lo cual resultaba muy lento con el aumento de tamaño. Esto condujo a la adopción de `Database Management Systems` (`DBMS`).

## Sistemas de gestión de bases de datos

Un Sistema de Gestión de Bases de Datos (SGBD) ayuda a crear, definir, alojar y gestionar bases de datos. A lo largo del tiempo se han diseñado diversos tipos de SGBD, como los basados ​​en archivos, los SGBD relacionales (SGBDR), los NoSQL, los basados ​​en grafos y los almacenes de clave-valor.

Existen múltiples maneras de interactuar con un SGBD, como herramientas de línea de comandos, interfaces gráficas o incluso API (Interfaces de Programación de Aplicaciones). Los SGBD se utilizan en diversos sectores bancarios, financieros y educativos para registrar grandes cantidades de datos. Algunas de las características esenciales de un SGBD incluyen:

| **Característica**          | **Descripción**                                                                                                                                                                            |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Concurrency`               | Una aplicación real puede tener varios usuarios interactuando simultáneamente. Un SGBD garantiza que estas interacciones concurrentes se realicen correctamente sin dañar ni perder datos. |
| `Consistency`               | Con tantas interacciones simultáneas, el DBMS debe garantizar que los datos permanezcan consistentes y válidos en toda la base de datos.                                                   |
| `Security`                  | El SGBD proporciona controles de seguridad detallados mediante la autenticación y los permisos de usuario. Esto evitará la visualización o edición no autorizada de datos confidenciales.  |
| `Reliability`               | Es fácil realizar copias de seguridad de bases de datos y revertirlas a un estado anterior en caso de pérdida o violación de datos.                                                        |
| `Structured Query Language` | SQL simplifica la interacción del usuario con la base de datos con una sintaxis intuitiva que admite diversas operaciones.                                                                 |

## Arquitectura

El diagrama a continuación detalla una arquitectura de dos niveles.

![Diagrama de arquitectura de tres niveles: El usuario interactúa con una aplicación cliente en el Nivel I, que se conecta a un servidor de aplicaciones en el Nivel II y a un SGBD en el Nivel III. Los usuarios y un administrador de base de datos acceden al SGBD.](../../.gitbook/assets/db_2.png)

`Tier I` generalmente consiste en aplicaciones del lado del cliente, como sitios web o programas con interfaz gráfica de usuario (GUI). Estas aplicaciones consisten en interacciones de alto nivel, como el inicio de sesión o los comentarios de los usuarios. Los datos de estas interacciones se transmiten a `Tier II` mediante llamadas a la API u otras solicitudes.

El segundo nivel es el middleware, que interpreta estos eventos y los presenta en el formato requerido por el SGBD. Finalmente, la capa de aplicación utiliza bibliotecas y controladores específicos según el tipo de SGBD para interactuar con ellos. El SGBD recibe consultas del segundo nivel y realiza las operaciones solicitadas. Estas operaciones pueden incluir la inserción, recuperación, eliminación o actualización de datos. Tras el procesamiento, el SGBD devuelve los datos solicitados o los códigos de error en caso de consultas no válidas.

Es posible alojar el servidor de aplicaciones y el SGBD en el mismo host. Sin embargo, las bases de datos con grandes cantidades de datos que admiten a muchos usuarios suelen alojarse por separado para mejorar el rendimiento y la escalabilidad.
