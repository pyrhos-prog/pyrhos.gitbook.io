---
icon: database
---

# Modelo entidad-relación

> Permite representar de forma grafica y conceptual los elementos antes de construir una base de datos.

**Para ello se utiliza:**

* **Entidades:** Representan los objetos de interes&#x20;
* **Atributos:** Describen las caracteristicas de las entidades
* **Relaciones:** Muestra como se asocian entre si

#### Pasos para realizar un modelo entidad relación

1. Se parte de una descripción textual del problema o sistema de información a automatizar (los requisitos).
2. Se hace una lista de los sustantivos y verbos que aparecen.
3. Los sustantivos son posibles entidades o atributos.
4. Los verbos son posibles relaciones.
5. Analizando las frases se determina la cardinalidad de las relaciones y otros detalles.
6. Se elabora el diagrama (o diagramas) entidad-relación.
7. Se completa el modelo con listas de atributos y una descripción de otras restricciones que no se pueden reflejar en el diagrama.



## Entidades

> Representa un objeto, concepto o elemento relevante del que queremos almacenar información.
>
> Debe de ser distinguible mediante algun método de identificación

Según su capacidad para identificarse por sí misma, podemos diferenciar entre **entidades fuertes** y **entidades débiles**.

### Entidades fuertes

Son entidades que se pueden identifcar por si mismas mediante uno o varios de sus propios atributos, sin necesidad de clave o de otra entidad.

Por ejemplo: en una base de datos de un centro educativo, una ocurrencia de la entidad **ALUMNO** puede identificarse mediante su propio identificador, independientemente de la entidad **CURSO**.                         &#x20;

### Entidades débiles

Son entidades que no pueden identificarse completamente mediante sus propios atributos y necesitan utilizar la clave de una entidad fuerte relacionada, se representan mediante un doble rectángulo.

**Presentan dos tipos de dependencia:**

* Dependencia en existencia: Una ocurrencia de una entidad dependiente no puede existir sin estar asociada a una ocurrencia de la entidad de la que depende. Si desaparece esta última, puede perder también su sentido la existencia de la primera.
* Dependencia en identificación: Además de existir dependencia, una ocurrencia de la entidad débil no puede identificarse únicamente mediante sus propios atributos, por lo que necesita incorporar la clave de la entidad fuerte asociada.

## Los Atributos

> Los atributos son las características o propiedades que describen a una Entidad o a una Relación. Toman sus valores de un dominio determinado.

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

```
Entidad: CLIENTES
Atributos: Código de Cliente, DNI, Apellidos, Nombre, Dirección, Teléfono.
```

### Dominio de un Atributo

El dominio es el conjunto de los posibles valores válidos que ese atributo puede poseer. Todos los valores que tome un atributo deben estar obligatoriamente dentro de su dominio. Varios atributos pueden estar definidos dentro del mismo dominio.

| **Tipo de Dominio**  | **Descripción**                                                       | **Ejemplo**                                     |
| -------------------- | --------------------------------------------------------------------- | ----------------------------------------------- |
| Cadenas de texto     | Cadenas de caracteres con determinadas restricciones.                 | `Nombre`, `Apellidos`                           |
| Formatos específicos | Cadenas que permiten almacenar patrones numéricos o de texto válidos. | `Teléfono`                                      |
| Valores restringidos | Lista cerrada de opciones predefinidas.                               | `Estado_Pedido` (Pendiente, Enviado, Entregado) |

### Clasificación de los Atributos

Haz clic en cada sección para expandir la información de cada clasificación:

* **Atributo obligatorio:** Es aquel que ha de estar siempre definido para una entidad o relación.
  * _**Ejemplo:**_ Para la entidad `CLIENTE`, su `DNI`. (Una clave o llave siempre es un atributo obligatorio).
* At**ributo opcional:** Es aquel que podría ser definido o no para la entidad. Puede haber ocurrencias (registros) para las que ese atributo no tenga valor.
  * _**Ejemplo:**_ El `Teléfono` fijo o un segundo email.
* **Atributo simple:** No se puede subdividir. Es un dato atómico.
  * _Ejemplo:_ El `Teléfono`.
* **Atributo compuesto:** Se puede subdividir en otros atributos más pequeños con significado propio.
  * _Ejemplo:_ La `Dirección` se puede descomponer en: `Calle`, `Número`, `Localidad`, `Provincia` y `Código Postal`.
* **Atributos monovaluados:** Tienen un único valor para cada ocurrencia de la entidad.
  * _Ejemplo:_ El `DNI` o la `Fecha de nacimiento` de una persona.
* **Atributos multivaluados:** Pueden tener varios valores independientes para una misma ocurrencia.
  * _**Ejemplo:**_ Los `Colores` de un vehículo, o las distintas `Fechas de matriculación` de un coche.

> En un modelo relacional normalizado no se almacenan varios valores en una única columna. Se suelen adoptar soluciones como:
>
> 1. Crear atributos diferentes si el número de valores es fijo (Ej: `Telefono_1`, `Telefono_2`).
> 2. Crear una nueva entidad o tabla relacionada que almacene cada uno de los valores.

* **Atributo derivado:** Es aquel cuyo valor no se almacena, sino que se deduce u obtiene calculándolo a partir de otro u otros atributos.
  * _**Ejemplo 1:**_ La `Edad` (se calcula restando la fecha actual a la fecha de nacimiento).
  * _**Ejemplo 2:**_ El `Importe` de una línea de pedido (se obtiene multiplicando `Precio_unitario` por `Cantidad`).

_Normalmente no es necesario almacenarlos, aunque a veces se "materializan" en el diseño físico por motivos de rendimiento del sistema._

## Las Claves en el Modelo Entidad-Relación

> Las claves (o llaves) son atributos o conjuntos de atributos que permiten identificar de forma única e inequívoca cada ocurrencia (registro) de una entidad. Garantizan que ningún par de entidades tengan exactamente los mismos valores en esos atributos identificadores.

### Tipos de Claves Naturales

Haz clic en cada sección para expandir la información de la evolución y selección de claves:

<details>

<summary><strong>1. Superclave (Superllave)</strong></summary>

Es cualquier conjunto de atributos que permite identificar de forma única una ocurrencia de entidad. _Nota: Una superclave puede contener atributos adicionales o innecesarios para realizar dicha identificación, no está optimizada._

</details>

<details>

<summary><strong>2. Clave Candidata</strong></summary>

Es una superclave mínima. Es decir, si eliminamos cualquiera de sus atributos, deja de permitir la identificación única de cada ocurrencia. Una entidad puede tener varias claves candidatas.

</details>

<details>

<summary><strong>3. Clave Primaria o Principal (Primary Key)</strong></summary>

De todas las claves candidatas, el diseñador de la base de datos escoge una, que se convertirá en la clave principal. Lo ideal es que esté formada por el menor número de atributos posible (preferiblemente uno solo).

**Requisitos indispensables:**

1. Sus valores deben ser únicos.
2. No puede contener valores nulos (vacíos).
3. Debe identificar de manera estable cada ocurrencia (su valor no debería cambiar con el tiempo).

</details>

<details>

<summary><strong>4. Claves Alternativas</strong></summary>

Son el resto de claves candidatas que cumplen todos los requisitos para identificar registros unívocamente, pero que finalmente no han sido escogidas como clave primaria.

</details>

### Ejemplo

```
Entidad: CLIENTES

* Superclaves posibles: CodigoCliente + Nombre, CodigoCliente + DNI...
* Claves candidatas: CodigoCliente y DNI (asumiendo que ambos sean únicos y no nulos).
* Clave primaria seleccionada: CodigoCliente.
* Clave alternativa: DNI (pasará a tener una restricción de valor único en la base de datos).
```



