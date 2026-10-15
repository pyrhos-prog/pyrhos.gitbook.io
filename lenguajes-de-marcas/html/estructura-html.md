---
icon: html5
---

# Estructura HTML

> **La estructura básica de HTML permite que el navegador identifique correctamente la página, sus metadatos y el contenido visible.**

### Plantilla base

Esto es el punto de partida de cualquier página:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Mi página web</title>
</head>
<body>
  <h1>Contenido principal</h1>
  <p>Este texto se muestra en el navegador.</p>
</body>
</html>
```

| Línea                        | Qué hace                                                                                               |
| ---------------------------- | ------------------------------------------------------------------------------------------------------ |
| `<!DOCTYPE html>`            | Le dice al navegador que use el estándar HTML actual. No referencia ninguna DTD ni versión             |
| `<html lang="es">`           | Elemento raíz. `lang` declara el idioma (mejora accesibilidad y pronunciación de lectores de pantalla) |
| `<head>`                     | Metadatos: no se muestran como contenido                                                               |
| `<meta charset="UTF-8">`     | Codificación. Sin ella los acentos pueden verse mal. Guarda el archivo también en UTF-8                |
| `<meta name="viewport" ...>` | Adaptación a móviles                                                                                   |
| `<title>`                    | Texto de la pestaña del navegador. Obligatorio en una página completa                                  |
| `<body>`                     | Todo lo visible: texto, imágenes, enlaces, tablas, formularios                                         |

> #### DOCTYPE históricos (ya no se usan)
>
> HTML 4.01 tenía tres variantes: **Strict** (sin elementos de presentación obsoletos), **Transitional** (admitía elementos antiguos) y **Frameset** (para marcos). Hoy `<frameset>` y `<frame>` están obsoletos.

### Anatomía de un elemento

```html
<a href="https://www.w3.org/" target="_blank">Ir al W3C</a>
<!--
  <a ...>   = etiqueta de apertura
  href, target = atributos (nombre="valor")
  Ir al W3C = contenido
  </a>      = etiqueta de cierre
-->
```

Un elemento tiene: etiqueta de apertura, **cero o más** atributos, contenido opcional y etiqueta de cierre.

Elementos vacíos (sin cierre ni contenido): `img`, `br`, `input`, `meta`, `link`, `base`.

```html
<img src="servidor.jpg" alt="Servidor instalado en un centro de datos">
```

Si un atributo tiene un valor inválido, el navegador suele ignorarlo o usar uno por defecto, sin avisar. Por eso se valida el código.

### Elementos de bloque y en línea

| Tipo     | Comportamiento                                          | Ejemplos              |
| -------- | ------------------------------------------------------- | --------------------- |
| Bloque   | Empieza en línea nueva y ocupa todo el ancho disponible | `h1`, `p`, `div`      |
| En línea | Ocupa solo lo necesario y sigue en la misma línea       | `a`, `strong`, `span` |

Esto es el comportamiento **por defecto**. Se puede cambiar con la propiedad CSS `display`. El estándar actual los clasifica en categorías de contenido (flujo, frase), pero para el módulo basta con la distinción bloque/línea.

### Atributos globales

Se pueden poner en casi cualquier elemento.

| Atributo      | Función                                                | Notas                                                              |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------------ |
| `id="x"`      | Identificador **único** en el documento                | Sirve para enlaces internos, CSS, JS y accesibilidad. Sin espacios |
| `class="a b"` | Agrupa elementos                                       | Un elemento puede tener varias clases separadas por espacios       |
| `style="..."` | CSS directo sobre el elemento                          | Válido, pero mejor una hoja externa                                |
| `title="..."` | Información adicional (tooltip en algunos navegadores) | No usar como único medio para info importante                      |
| `data-*`      | Datos personalizados                                   | `data-codigo="P104"`                                               |
| `lang="es"`   | Idioma del documento o de un fragmento                 | Códigos BCP 47: `es`, `en-GB`, `fr-CA`, `ja`                       |
| `dir="ltr"`   | Dirección del texto                                    | `ltr`, `rtl`, `auto`                                               |
| `xml:lang`    | Idioma en documentos XML/XHTML                         | En HTML normal usa `lang`                                          |

`name` **no** es global: solo existe en formularios, controles y algunos metadatos.

```html
<html lang="es">
  <p>Contenido en español.</p>
  <p lang="en">Content in English.</p>
</html>
```

Los atributos de eventos (`onclick`) y `tabindex` (foco y teclado) se ven con JavaScript y accesibilidad.

### Errores frecuentes

| Error                                                     | Corrección                                   |
| --------------------------------------------------------- | -------------------------------------------- |
| Escribir contenido visible dentro de `<head>`             | Va en `<body>`                               |
| Olvidar `<meta charset="UTF-8">`                          | Acentos rotos                                |
| Repetir el mismo `id`                                     | Cada `id` es único; para repetir usa `class` |
| `<img>` sin `alt`                                         | Siempre `alt`                                |
| Usar `<center>` o `<font>`                                | CSS                                          |
| Guardar el archivo en otra codificación distinta de UTF-8 | Guardar en UTF-8                             |
