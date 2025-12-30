# Tipos de bases de datos

Las bases de datos, en general, se clasifican en \[nombre del sistema] `Relational Databases` y `Non-Relational Databases` \[nombre del sistema]. Solo las bases de datos relacionales utilizan SQL, mientras que las no relacionales utilizan diversos métodos de comunicación.

## Bases de datos relacionales

Una base de datos relacional es el tipo más común. Utiliza un esquema, una plantilla, para determinar la estructura de datos almacenada. Por ejemplo, imaginemos que una empresa que vende productos a sus clientes tiene algún tipo de información almacenada sobre dónde se envían esos productos, a quién se los envían y en qué cantidad. Sin embargo, esto suele hacerse en el backend y sin información clara en el frontend. Se pueden utilizar diferentes tipos de bases de datos relacionales para cada enfoque. Por ejemplo, la primera tabla puede almacenar y mostrar información básica del cliente, la segunda, el número de productos vendidos y su coste, y la tercera, enumerar quién compró esos productos y con qué datos de pago.

Las tablas de una base de datos relacional están asociadas a claves que proporcionan un resumen rápido de la base de datos o acceso a la fila o columna específica cuando es necesario revisar datos específicos. Estas tablas, también llamadas entidades, están relacionadas entre sí. Por ejemplo, la tabla de información del cliente puede proporcionar a cada cliente un ID específico que indica toda la información necesaria sobre él, como su dirección, nombre e información de contacto. Asimismo, la tabla de descripción del producto puede asignar un ID específico a cada producto. La tabla que almacena todos los pedidos solo necesita registrar estos ID y su cantidad. Cualquier cambio en estas tablas las afectará a todas, pero de forma predecible y sistemática.

Sin embargo, al procesar una base de datos integrada, se requiere un concepto para vincular una tabla con otra mediante su clave, denominada `relational database management system` (`RDBMS`). Muchas empresas que inicialmente utilizaban conceptos diferentes están adoptando el concepto de RDBMS porque es fácil de aprender, usar y comprender. Inicialmente, este concepto solo lo utilizaban las grandes empresas. Sin embargo, ahora muchos tipos de bases de datos implementan el concepto de RDBMS, como Microsoft Access, MySQL, SQL Server, Oracle, PostgreSQL y muchas otras.

Por ejemplo, podemos tener una `users` tabla en una base de datos relacional que contenga columnas como `id`, `username`, `first_name`, `last_name`, y otras. \[El/La /Los/ `id`Las] puede usarse como clave de la tabla. Otra tabla, `posts`, puede contener publicaciones de todos los usuarios, con columnas como `id`, `user_id`, `date`, `content`, , etc.

![Diagrama de esquema de base de datos que muestra la tabla "usuarios" con las columnas: id, nombre de usuario, nombre, apellido, y la tabla "publicaciones" con las columnas: id, id de usuario, fecha y contenido. Las flechas indican la relación entre el id de usuario en "publicaciones" y el id en "usuarios".](../../.gitbook/assets/web_apps_relational_db.jpg)

Podemos vincular "`id` de la `users` tabla" con "`user_id` dentro de la `posts` tabla" para recuperar los datos de usuario de cada publicación sin almacenarlos todos en cada una. Una tabla puede tener más de una clave, ya que cada columna puede usarse como clave para vincularse con otra tabla. Por ejemplo, la `id` columna puede usarse como clave para vincular la `posts` tabla con otra tabla que contenga comentarios, cada uno de los cuales pertenece a una publicación específica, y así sucesivamente.

La relación entre las tablas dentro de una base de datos se llama esquema.

De esta manera, al usar bases de datos relacionales, es rápido y sencillo recuperar todos los datos sobre un elemento específico de todas las bases de datos. Por ejemplo, podemos recuperar todos los detalles vinculados a un usuario específico de todas las tablas con una sola consulta. Esto hace que las bases de datos relacionales sean muy rápidas y confiables para grandes conjuntos de datos, con una estructura y un diseño claros y una gestión de datos eficiente. El ejemplo más común de bases de datos relacionales es \[texto incoherente] `MySQL`, que abordaremos en este módulo.

## Bases de datos no relacionales

Una base de datos no relacional (también llamada `NoSQL` base de datos) no utiliza tablas, filas y columnas, ni claves principales, relaciones ni esquemas. En cambio, una base de datos NoSQL almacena datos mediante diversos modelos de almacenamiento, según el tipo de datos almacenados. Debido a la falta de una estructura definida, las bases de datos NoSQL son muy escalables y flexibles. Por lo tanto, al trabajar con conjuntos de datos poco definidos y estructurados, una base de datos NoSQL es la mejor opción para almacenar dichos datos. Existen cuatro modelos de almacenamiento comunes para bases de datos NoSQL:

* Clave-valor
* Basado en documentos
* Columna ancha
* Gráfico

Cada uno de los modelos anteriores almacena datos de forma diferente. Por ejemplo, el `Key-Value` modelo suele almacenar datos en JSON o XML, tiene una clave para cada par y almacena todos sus datos como su valor:

![Tabla de publicaciones con entradas: id 100001, fecha 01-01-2021, contenido «Bienvenido a esta aplicación web»; id 100002, fecha 02-01-2021, contenido «Esta es la primera publicación de esta aplicación web»; id 100003, fecha 02-01-2021, contenido «Recordatorio: Mañana es el...». Se indican las claves y los valores.](<../../.gitbook/assets/web_apps_non relational_db.jpg>)

El ejemplo anterior se puede representar utilizando JSON como:

{% code title="ejemplo.json" %}
```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```
{% endcode %}

Parece similar a un elemento de diccionario en lenguajes como `Python` o `PHP` (es decir, `{'key':'value'}`), donde `key` generalmente es una cadena y `value` puede ser una cadena, un diccionario o cualquier objeto de clase.

El ejemplo más común de una base de datos NoSQL es `MongoDB`.

Las bases de datos no relacionales utilizan un método de inyección diferente, conocido como inyección NoSQL. Las inyecciones SQL son completamente diferentes a las inyecciones NoSQL. Las inyecciones NoSQL se abordarán en un módulo posterior.
