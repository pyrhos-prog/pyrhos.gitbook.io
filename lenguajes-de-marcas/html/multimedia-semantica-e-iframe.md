---
icon: html5
---

# Multimedia, semántica e iframe

### Multimedia

| Elemento | Función                                                          |
| -------- | ---------------------------------------------------------------- |
| `img`    | Inserta una imagen                                               |
| `audio`  | Incorpora audio                                                  |
| `video`  | Reproduce vídeo                                                  |
| `object` | Incrusta recursos externos (por ejemplo, PDF). Poco habitual hoy |

#### Imágenes: `<img>`

Elemento vacío (sin cierre).

| Atributo          | Función                                                  |
| ----------------- | -------------------------------------------------------- |
| `src`             | Ruta o URL de la imagen                                  |
| `alt`             | Texto alternativo: lectores de pantalla y fallo de carga |
| `width`, `height` | Dimensiones. Reservan el espacio antes de que cargue     |
| `loading="lazy"`  | Retrasa la carga de imágenes lejos de la zona visible    |

```html
<!-- Imagen en internet -->
<img src="https://ejemplo.com/luna.jpg" alt="La luna llena">

<!-- Imagen local (misma carpeta) -->
<img src="logo.jpeg" alt="Logotipo de la empresa" width="200" height="80">

<!-- Imagen en carga diferida (galerías largas) -->
<img src="foto25.jpg" alt="Foto 25 de la galería" loading="lazy">
```

Reglas para `alt`:

| Imagen             | `alt`                                                              |
| ------------------ | ------------------------------------------------------------------ |
| Aporta información | Descripción breve de su contenido o finalidad                      |
| Solo decorativa    | `alt=""` (vacío) para que las tecnologías de asistencia la ignoren |
| Falta el atributo  | Error de accesibilidad                                             |

{% hint style="info" %}
Formatos habituales: JPEG, PNG, GIF, y también WebP, AVIF y SVG. El tamaño adaptable se controla con CSS (`max-width: 100%`).
{% endhint %}

#### Audio y vídeo

Esto es lo mínimo para usarlos:

```html
<audio controls src="pista.mp3">Tu navegador no admite audio.</audio>

<video controls width="480" poster="portada.jpg">
  <source src="clip.mp4" type="video/mp4">
  <source src="clip.webm" type="video/webm">
  Tu navegador no admite vídeo.
</video>
```

`controls` muestra los controles de reproducción. Con varios `source`, el navegador usa el primero que soporte. El texto interior es el contenido alternativo.

### Etiquetas semánticas

Describen la **función** de cada bloque en lugar de ser contenedores genéricos. Mejoran accesibilidad, mantenimiento y comprensión por parte de navegadores y buscadores.

| Elemento  | Función                                                                           |
| --------- | --------------------------------------------------------------------------------- |
| `header`  | Contenido introductorio de la página o sección: título, logo                      |
| `nav`     | Enlaces de navegación principales                                                 |
| `main`    | Contenido principal y exclusivo de la página                                      |
| `section` | Sección temática, normalmente con un encabezado                                   |
| `article` | Contenido independiente y reutilizable: noticia, entrada, publicación             |
| `aside`   | Información relacionada de forma secundaria. No tiene por qué estar en un lateral |
| `footer`  | Información final: autoría, enlaces legales, contacto                             |
| `div`     | Agrupación **sin** significado, para cuando no hay etiqueta mejor                 |

Ejemplo de página de noticias:

```html
<body>
  <header>
    <h1>Diario Tecnológico</h1>
  </header>

  <nav>
    <a href="index.html">Inicio</a>
    <a href="contacto.html">Contacto</a>
  </nav>

  <main>
    <article>
      <h2>Nueva versión del estándar</h2>
      <p>Texto de la noticia...</p>
    </article>

    <aside>
      <h2>Relacionado</h2>
      <p>Otras noticias...</p>
    </aside>
  </main>

  <footer>
    <p>Aviso legal y contacto</p>
  </footer>
</body>
```

Importante: la **posición y el aspecto** de estos bloques no dependen de la etiqueta, sino de las reglas CSS.

Elegir bien:

| Necesito                           | Uso                                |
| ---------------------------------- | ---------------------------------- |
| Cabecera con logo y título         | `header`                           |
| Menú principal                     | `nav`                              |
| Una noticia o entrada de blog      | `article`                          |
| Información secundaria relacionada | `aside`                            |
| Datos legales y de contacto        | `footer`                           |
| Agrupar solo para aplicar estilo   | `div` (bloque) o `span` (en línea) |

Error típico: envolver toda la página en `section`, o usar solo `div`, cuando existen etiquetas más específicas.

### `<iframe>`: marcos integrados

Muestra una página o recurso externo dentro de otra. Sustituye en algunos usos a los antiguos `frame` y `frameset`, eliminados del HTML actual.

| Atributo          | Función                                                   |
| ----------------- | --------------------------------------------------------- |
| `src`             | Dirección del contenido inicial                           |
| `name`            | Nombre del marco, para usarlo como destino de enlaces     |
| `width`, `height` | Dimensiones (mejor con CSS)                               |
| `title`           | Describe el contenido. Mejora la accesibilidad            |
| `loading="lazy"`  | Retrasa la carga hasta que está cerca de la zona visible  |
| `sandbox`         | Aplica restricciones de seguridad al contenido incrustado |

#### Menú que carga páginas dentro del marco

El `target` del enlace debe coincidir con el `name` del `iframe`:

```html
<nav>
  <a href="listas.html" target="contenido">Ejemplo de listas</a>
  <a href="tablas.html" target="contenido">Ejemplo de tablas</a>
</nav>

<iframe
  src="listas.html"
  name="contenido"
  title="Ejemplos de HTML"
  width="600"
  height="350"
  loading="lazy">
</iframe>
```

Al inicio se ve `listas.html`. Al pulsar otro enlace se carga dentro del mismo marco, sin reemplazar toda la página.

#### Insertar un mapa o recurso externo con restricciones

```html
<iframe
  src="https://ejemplo.com/mapa"
  title="Mapa de la ubicación de la oficina"
  width="600"
  height="400"
  loading="lazy"
  sandbox>
</iframe>
```

Seguridad y límites:

* Un `iframe` crea un contexto de navegación independiente. No incrustes contenido desconocido sin restricciones: usa `sandbox`.
* Algunas webs impiden ser incrustadas mediante políticas de seguridad: el marco aparecerá vacío o con error.
* Sin `title`, el marco es poco accesible.

### Errores frecuentes

| Error                                       | Corrección                             |
| ------------------------------------------- | -------------------------------------- |
| `img` sin `alt`                             | Siempre `alt` (vacío si es decorativa) |
| `alt` largo en un icono decorativo          | `alt=""`                               |
| Usar `div` para todo                        | Etiquetas semánticas                   |
| `iframe` sin `title`                        | Añadir `title`                         |
| `target` que no coincide con `name`         | Deben ser iguales                      |
| Incrustar sitios desconocidos sin `sandbox` | Añadir restricciones                   |
| Usar `frameset`                             | Obsoleto: HTML semántico + CSS         |
