---
icon: html5
---

# Cabeceras, textos enlaces y listas

### Elementos de `<head>`

El `<head>` contiene metadatos y recursos. Nada de lo que pongas aquí se muestra como contenido de la página (salvo el `<title>` en la pestaña).

| Elemento   | Tipo       | Función                                                           |
| ---------- | ---------- | ----------------------------------------------------------------- |
| `title`    | Contenedor | Título de la pestaña. Obligatorio en una página completa          |
| `script`   | Contenedor | Incluye o enlaza JavaScript                                       |
| `style`    | Contenedor | Reglas CSS internas del documento                                 |
| `noscript` | Contenedor | Contenido alternativo si JavaScript no está disponible            |
| `meta`     | Vacío      | Codificación, descripción, autor, adaptación a móviles            |
| `link`     | Vacío      | Relaciona con recursos externos: hojas de estilo, iconos, fuentes |
| `base`     | Vacío      | URL base para resolver enlaces relativos                          |

`isindex` está obsoleto y eliminado. Para pedir datos al usuario se usan formularios (`form` + `input`).

Cabecera típica completa:

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Página de ejemplo de la unidad 2">
  <meta name="author" content="Tu nombre">
  <title>Mi página</title>
  <link rel="stylesheet" href="estilos.css">
</head>
```

### Encabezados y párrafos

```html
<h1>Equipos</h1>
<p>Primer párrafo.</p>
<p>Segundo párrafo.</p>
<h2>Recién ascendidos</h2>
<p>...</p>
<h1>Jugadores</h1>
<p>...</p>
```

| Elemento    | Detalle                                                 |
| ----------- | ------------------------------------------------------- |
| `h1` a `h6` | Seis niveles. Cuanto menor el número, mayor importancia |
| `p`         | Párrafo. Elemento de bloque                             |

Reglas de uso:

* Los encabezados reflejan la **jerarquía** del contenido, no el tamaño. Si quieres letra grande, usa CSS.
* No saltes niveles (de `h1` directamente a `h4`): HTML lo muestra, pero rompe la jerarquía para lectores de pantalla y buscadores.
* Ten un `h1` como título principal.

### Espacios y saltos de línea

El navegador **colapsa** varios espacios seguidos en uno e **ignora** la mayoría de saltos de línea del código fuente.

| Necesito                          | Uso                    | Nota                                                           |
| --------------------------------- | ---------------------- | -------------------------------------------------------------- |
| Salto de línea dentro de un texto | `<br>`                 | Elemento vacío                                                 |
| Espacio inseparable               | `&nbsp;`               | No lo uses para crear márgenes o separar elementos: eso es CSS |
| Separar bloques                   | `<p>` o CSS (`margin`) |                                                                |

```html
Esto
se
ve
en una línea sin saltos.
<p>Aquí hay &nbsp;&nbsp;&nbsp; espacios inseparables.</p>
Esto <br> introduce un salto de línea.
```

### Comentarios

```html
<!-- comentario de una línea -->

<!-- comentario
     de varias líneas -->
```

No se muestran en la página, pero **se ven en el código fuente** (clic derecho, "Ver código fuente"). Nunca pongas contraseñas, datos personales ni información confidencial en ellos.

### Semántica a nivel de texto

| Elemento | Significado                          | Cuándo                             |
| -------- | ------------------------------------ | ---------------------------------- |
| `strong` | Especial importancia                 | Avisos, términos críticos          |
| `em`     | Énfasis                              | Palabra que se lee con intención   |
| `small`  | Información secundaria               | Avisos legales, anotaciones        |
| `br`     | Salto de línea                       | Direcciones, versos                |
| `a`      | Hipervínculo                         | Enlaces                            |
| `span`   | Sin significado, contenedor en línea | Solo cuando no hay etiqueta mejor) |

```html
Texto marcado como <strong>importante</strong>.<br>
Texto con <em>énfasis</em>.<br>
Texto <small>secundario</small>.<br>
Pulsa <a href="https://www.w3.org/">aquí</a> para ir al W3C.
```

Regla práctica: si el texto es importante usa `strong`, no `span` con estilo.

### Enlaces: `<a>`

El atributo principal es `href` (destino). Sin `href` el elemento no funciona como enlace.

| Tipo de destino                    | Ejemplo                                 |
| ---------------------------------- | --------------------------------------- |
| URL absoluta                       | `<a href="https://www.w3.org/">W3C</a>` |
| Archivo relativo                   | `<a href="contacto.html">Contacto</a>`  |
| Sección de la misma página (ancla) | `<a href="#contacto">Ir a contacto</a>` |
| Inicio de la página                | `<a href="#">Arriba</a>`                |

Para que el ancla funcione, el destino tiene un `id`:

```html
<a href="#contacto">Ir a contacto</a>
...
<h2 id="contacto">Contacto</h2>
```

Atributo `target`:

| Valor                 | Efecto                              |
| --------------------- | ----------------------------------- |
| `_self`               | Misma pestaña (por defecto)         |
| `_blank`              | Pestaña nueva                       |
| nombre de un `iframe` | Carga el enlace dentro de ese marco |

Con `_blank`, añade `rel="noopener"` para que la página abierta no pueda acceder a la de origen:

```html
<a href="https://ejemplo.com" target="_blank" rel="noopener">Enlace externo seguro</a>
```

### Listas

| Elemento | Función                                                   |
| -------- | --------------------------------------------------------- |
| `ul`     | Lista desordenada (viñetas)                               |
| `ol`     | Lista ordenada (numerada). Atributos: `start`, `reversed` |
| `li`     | Cada elemento de `ul` u `ol`                              |
| `dl`     | Lista de definiciones                                     |
| `dt`     | Término                                                   |
| `dd`     | Descripción del término                                   |

Cuándo usar cada una:

* Pasos de instalación o procedimiento: `ol` (el orden importa).
* Conjunto sin orden: `ul`.
* Término y explicación: `dl`. No simules definiciones con una `ul`.

```html
<h2>Módulos de 1º de ASIR</h2>
<ul>
  <li>Fundamentos de Hardware</li>
  <li>Gestión de Bases de Datos</li>
</ul>

<ol start="4">
  <li>Empieza en el 4</li>
  <li>Sigue en el 5</li>
</ol>

<dl>
  <dt>Fundamentos de Hardware</dt>
  <dd>Componentes físicos de un ordenador</dd>
  <dt>Gestión de Bases de Datos</dt>
  <dd>Diseño y uso de bases de datos relacionales</dd>
</dl>
```

Complemento: una lista dentro de un `li` crea una lista anidada.

```html
<ul>
  <li>Redes
    <ul>
      <li>Switching</li>
      <li>Routing</li>
    </ul>
  </li>
  <li>Sistemas</li>
</ul>
```

El estilo de los marcadores (círculo, cuadrado, números romanos, imagen) se cambia con CSS.

### Errores frecuentes

| Error                                      | Corrección                                |
| ------------------------------------------ | ----------------------------------------- |
| Usar `<h1>` para todo lo que quiero grande | Encabezados por jerarquía, tamaño con CSS |
| Muchos `&nbsp;` o `<br>` para separar      | `margin` / `padding`                      |
| `href` ausente                             | Sin él no hay enlace                      |
| `target="_blank"` sin `rel="noopener"`     | Añadir `rel="noopener"`                   |
| Datos sensibles en comentarios             | Nunca                                     |
| Lista `ul` para términos con definición    | Usar `dl`                                 |

