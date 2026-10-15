---
icon: html5
---

# Formularios

> Un formulario recoge datos del usuario y los envía a una dirección para que se procesen.&#x20;

### Estructura de un formulario

```html
<form action="/procesar.php" method="post">
  <label for="usuario">Usuario</label>
  <input type="text" id="usuario" name="usuario" required>

  <button type="submit">Enviar</button>
</form>
```

#### Atributos de `<form>`

| Atributo  | Función                                                  |
| --------- | -------------------------------------------------------- |
| `action`  | Dirección que recibe y procesa los datos                 |
| `method`  | Método de envío: `get` o `post`                          |
| `enctype` | Codificación de los datos. Necesario para subir archivos |
| `name`    | Nombre del formulario (hoy se prefiere `id`)             |

#### GET vs POST

|                        | GET                                                 | POST                                                         |
| ---------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| Dónde viajan los datos | En la URL como parámetros                           | En el cuerpo de la petición                                  |
| Uso adecuado           | Búsquedas, filtros, consultas que no modifican nada | Registros, login, subida de archivos, cambios en el servidor |
| Visibilidad            | Se ven en la URL y quedan en el historial           | No aparecen en la URL                                        |
| Marcador / compartir   | Se puede guardar y compartir la URL                 | Normalmente no se reproduce solo con la dirección            |
| Tamaño                 | Limitado por navegador, servidor y URL              | Mayor, sujeto a límites del servidor                         |

Ejemplo de URL generada con GET:

```
https://ejemplo.com/buscar?categoria=portatiles&marca=lenovo
```

Puntos clave:

* **Nunca** envíes datos sensibles por GET.
* POST **no cifra** nada por sí mismo. Tanto GET como POST necesitan **HTTPS** para proteger la transmisión.
* Para subir archivos: `method="post"` + `enctype="multipart/form-data"`.

```html
<form action="/subir-archivo" method="post" enctype="multipart/form-data">
  <label for="doc">Documento</label>
  <input type="file" id="doc" name="doc">
  <button type="submit">Subir</button>
</form>
```

{% hint style="warning" %}
Todo control cuyo dato deba llegar al servidor necesita atributo `name`. Es la clave con la que se identifica el dato en la petición.
{% endhint %}

### `<input>`

Elemento vacío (sin cierre). Su comportamiento depende de `type`.

#### Atributos habituales

| Atributo      | Función                                                    |
| ------------- | ---------------------------------------------------------- |
| `type`        | Tipo de control (`text`, `password`, `radio`...)           |
| `name`        | Identifica el dato al enviar                               |
| `id`          | Para asociar con `label`                                   |
| `value`       | Valor inicial                                              |
| `size`        | Anchura visible aproximada, en caracteres                  |
| `maxlength`   | Máximo de caracteres                                       |
| `placeholder` | Sugerencia mientras está vacío. **No sustituye** a `label` |
| `readonly`    | No se puede modificar, pero **sí se envía**                |
| `autofocus`   | Foco al cargar la página                                   |
| `required`    | Obligatorio para poder enviar                              |

```html
<label for="usuario">Nombre de usuario</label>
<input
  type="text"
  id="usuario"
  name="usuario"
  size="30"
  maxlength="20"
  placeholder="Escriba su nombre de usuario"
  required
>
```

#### Tipos de `input`

| Tipo             | Para qué             | Notas                                                           |
| ---------------- | -------------------- | --------------------------------------------------------------- |
| `text`           | Texto de una línea   |                                                                 |
| `password`       | Contraseña           | Solo oculta los caracteres en pantalla. Enviar por HTTPS y POST |
| `email`          | Correo               | Comprueba formato básico de dirección                           |
| `url`            | Dirección web        | Comprueba formato válido de URL                                 |
| `number`         | Número               | Admite `min`, `max`, `step`                                     |
| `range`          | Deslizador           | Admite `min`, `max`, `step`, `value`                            |
| `file`           | Archivo              | Requiere POST y `multipart/form-data`                           |
| `hidden`         | Valor oculto         | No guardar información sensible                                 |
| `checkbox`       | Casilla              | Varias opciones independientes                                  |
| `radio`          | Botón de opción      | Una opción entre varias                                         |
| `date`           | Día, mes y año       |                                                                 |
| `month`          | Mes y año            |                                                                 |
| `week`           | Semana del año       |                                                                 |
| `time`           | Hora                 |                                                                 |
| `datetime-local` | Fecha y hora local   | Sin zona horaria                                                |
| `submit`         | Botón de envío       |                                                                 |
| `reset`          | Botón de restablecer |                                                                 |

#### Número

```html
<label for="edad">Edad</label>
<input type="number" id="edad" name="edad" min="16" max="99" step="1">
```

#### Fechas y horas

`min`, `max` y `step` también valen aquí. En `time` y `datetime-local`, `step` se expresa en **segundos**.

```html
<!-- Citas de septiembre de 2026, de 08:00 a 20:00, en tramos de 15 minutos -->
<label for="cita">Fecha y hora de la cita</label>
<input
  type="datetime-local"
  id="cita"
  name="cita"
  min="2026-09-01T08:00"
  max="2026-09-30T20:00"
  step="900"
>
```

`900` segundos = 15 minutos. El aspecto del selector cambia según navegador y sistema.

#### Rango

```html
<label for="volumen">Volumen</label>
<input type="range" id="volumen" name="volumen" min="0" max="100" step="5" value="50">
```

El deslizador no muestra siempre el número. Se complementa con JavaScript o con un elemento `<output>`.

#### Casilla (checkbox)

```html
<input type="checkbox" id="conforme" name="conforme" value="ok" checked>
<label for="conforme">Estoy conforme con la política de privacidad</label>
```

| Atributo   | Función                                     |
| ---------- | ------------------------------------------- |
| `checked`  | Marcada inicialmente                        |
| `value`    | Valor que se envía **solo si está marcada** |
| `required` | Hay que marcarla para enviar                |

Una casilla sin marcar **no aparece** en los datos enviados.

#### Botón de opción (radio)

Los radios con el **mismo `name`** forman un grupo y solo se puede elegir uno:

```html
<input type="radio" id="so-linux" name="so" value="linux" checked>
<label for="so-linux">Linux</label>

<input type="radio" id="so-win" name="so" value="windows">
<label for="so-win">Windows</label>
```

Casilla vs radio: varias opciones independientes, `checkbox`; una sola entre varias alternativas, `radio`.

### `<textarea>`

Texto de varias líneas. A diferencia de `input`, **sí** lleva cierre, y el texto inicial va entre las etiquetas.

```html
<label for="intereses">Describa sus intereses</label><br>
<textarea
  id="intereses"
  name="intereses"
  rows="5"
  maxlength="500"
  placeholder="Escriba aquí sus intereses"
  required
></textarea>
```

| Atributo                                                                | Función                          |
| ----------------------------------------------------------------------- | -------------------------------- |
| `rows`                                                                  | Filas visibles                   |
| `cols`                                                                  | Anchura aproximada en caracteres |
| `maxlength`, `required`, `placeholder`, `readonly`, `autofocus`, `name` | Igual que en `input`             |

No existe `value` en `textarea`. `rows` y `cols` siguen siendo válidos, pero hoy el tamaño se controla con `width` y `height` en CSS.

### Lista desplegable: `<select>` y `<option>`

```html
<label for="so">Sistema operativo favorito</label>
<select id="so" name="so">
  <option value="linux">Linux</option>
  <option value="windows">Windows</option>
  <option value="macos" selected>Mac OS</option>
</select>
```

| Elemento | Atributo   | Función                                             |
| -------- | ---------- | --------------------------------------------------- |
| `select` | `name`     | Identifica el dato al enviar                        |
| `select` | `size`     | Cuántas opciones se ven a la vez                    |
| `select` | `multiple` | Permite elegir varias (Ctrl, Cmd o gestos táctiles) |
| `option` | `value`    | Valor que se envía                                  |
| `option` | `selected` | Opción marcada inicialmente                         |

Ventaja frente a un campo de texto: valores controlados y menos errores de escritura. Para elección única, **no** pongas `multiple`.

### Botones

```html
<input type="submit" value="Enviar formulario">
<button type="submit">Enviar</button>
<input type="reset" value="Restablecer formulario">
```

| Botón    | Función                                                             |
| -------- | ------------------------------------------------------------------- |
| `submit` | Envía los datos a `action` con el `method` indicado                 |
| `reset`  | Devuelve los campos a sus **valores iniciales** (no siempre vacíos) |

`<button>` es más flexible porque admite contenido dentro (texto, iconos). Usa `reset` con cuidado: un clic accidental borra lo que el usuario escribió.

### Organización: `label`, `fieldset`, `legend`

#### `label`

Asocia un texto a un control. Se puede hacer de dos formas:

```html
<!-- 1) for = id del control -->
<label for="email">Correo</label>
<input type="email" id="email" name="email">

<!-- 2) envolviendo el control -->
<label>
  Correo
  <input type="email" name="email">
</label>
```

{% hint style="info" %}
Ventajas: al pulsar el texto se activa el control (útil en casillas y radios) y los lectores de pantalla identifican el campo. El `for` debe coincidir con un `id` **único**.
{% endhint %}

#### `fieldset` y `legend`

`fieldset` agrupa controles relacionados; `legend` es su título y debe ser el **primer** contenido del `fieldset`.

```html
<fieldset>
  <legend>Preferencias de contacto</legend>
  <label>
    <input type="checkbox" name="contacto" value="email">
    Correo electrónico
  </label>
  <label>
    <input type="checkbox" name="contacto" value="telefono">
    Teléfono
  </label>
</fieldset>
```

### Validación

Tres capas, que se combinan:

| Capa       | Herramientas                                                                                   | Papel                                  |
| ---------- | ---------------------------------------------------------------------------------------------- | -------------------------------------- |
| HTML       | `required`, `minlength`, `maxlength`, `pattern`, `type="email"`, `type="number"`, `min`, `max` | Comprobaciones básicas en el navegador |
| JavaScript | Validación propia, mensajes personalizados                                                     | Mejor experiencia de uso               |
| Servidor   | Revalidar todo                                                                                 | Seguridad e integridad                 |

La validación del navegador **no sustituye** a la del servidor: el cliente puede modificar o saltarse esas comprobaciones.

Ejemplo con `pattern` (código postal español de 5 dígitos):

```html
<label for="cp">Código postal</label>
<input
  type="text"
  id="cp"
  name="cp"
  pattern="[0-9]{5}"
  title="Cinco dígitos"
  required
>
```

### Formulario completo de ejemplo

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Formulario de registro</title>
</head>
<body>
  <h1>Registro</h1>
  <form action="/registro" method="post">

    <fieldset>
      <legend>Datos personales</legend>

      <label for="nombre">Nombre</label>
      <input type="text" id="nombre" name="nombre" maxlength="30" required>
      <br>

      <label for="correo">Correo</label>
      <input type="email" id="correo" name="correo" required>
      <br>

      <label for="clave">Contraseña</label>
      <input type="password" id="clave" name="clave" minlength="8" required>
      <br>

      <label for="edad">Edad</label>
      <input type="number" id="edad" name="edad" min="16" max="99">
    </fieldset>

    <fieldset>
      <legend>Preferencias</legend>

      <label for="so">Sistema operativo</label>
      <select id="so" name="so">
        <option value="linux">Linux</option>
        <option value="windows">Windows</option>
        <option value="macos">Mac OS</option>
      </select>
      <br>

      <label>
        <input type="checkbox" name="intereses" value="redes"> Redes
      </label>
      <label>
        <input type="checkbox" name="intereses" value="bd"> Bases de datos
      </label>
      <br>

      <label for="comentario">Comentarios</label><br>
      <textarea id="comentario" name="comentario" rows="4" cols="40"></textarea>
    </fieldset>

    <label>
      <input type="checkbox" name="privacidad" value="ok" required>
      Acepto la política de privacidad
    </label>
    <br>

    <button type="submit">Registrarme</button>
    <button type="reset">Borrar</button>
  </form>
</body>
</html>
```

### Errores frecuentes

| Error                                          | Consecuencia                      | Corrección                       |
| ---------------------------------------------- | --------------------------------- | -------------------------------- |
| Control sin `name`                             | El dato no se envía               | Poner `name`                     |
| Usar `placeholder` en lugar de `label`         | Campo poco claro y poco accesible | `label` + `placeholder` opcional |
| Contraseña por GET o por HTTP                  | Se expone en URL, historial o red | POST + HTTPS                     |
| Dar por válido solo lo que valida el navegador | Datos manipulables                | Validar en servidor              |
| `value` en `textarea`                          | No funciona                       | Texto entre etiquetas            |
| `multiple` para elección única                 | Permite varias                    | Quitar `multiple`                |
| Repetir `id` en varios `label`/controles       | `for` ambiguo                     | `id` único por control           |
| Radios con distinto `name`                     | No forman grupo                   | Mismo `name`                     |
| `readonly` creyendo que no se envía            | Sí se envía                       | Saber la diferencia              |
