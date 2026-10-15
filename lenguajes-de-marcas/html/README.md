---
icon: html5
---

# HTML

> HTML (HyperText Markup Language) define la **estructura y el significado** del contenido de una página. No sirve para dar aspecto (eso es CSS) ni para programar comportamiento (eso es JavaScript).

**HTML se basa en:**

* **Texto plano:** Los documentos podían crearse y transferirse entre diferentes equipos mediante codificaciones de caracteres como ASCII.
* **SGML:** Proporcionaba las bases para definir etiquetas capaces de identificar títulos, párrafos, enlaces y otros elementos del documento.

**HTML tiene limitaciones:**

* Por sí solo, ofrece pocas opciones de diseño y presentación; estas tareas se realizan principalmente con **CSS**.
* Dispone de un conjunto de elementos definido y no permite crear nuevas etiquetas con significado propio de forma libre.
* No genera contenido dinámico sin la ayuda de JavaScript o tecnologías del servidor.
* Una mala utilización puede mezclar estructura, presentación y comportamiento, dificultando el mantenimiento del documento.

### Separación de responsabilidades

| Tecnología | Se encarga de                          | Ejemplo                                    |
| ---------- | -------------------------------------- | ------------------------------------------ |
| HTML       | Estructura y significado               | `<header>`, `<nav>`, `<main>`, `<article>` |
| CSS        | Colores, tamaños, fondos, distribución | `color: #333;`                             |
| JavaScript | Interacción y comportamiento dinámico  | Un botón que actualiza contenido           |

Consecuencia práctica: `<font>`, `<center>` y el atributo `align` están **obsoletos**. El aspecto se define siempre con CSS.

### Evolución

| Versión              | Fecha                    | Dato clave                                                                                  |
| -------------------- | ------------------------ | ------------------------------------------------------------------------------------------- |
| HTML 2.0             | Nov 1995                 | Primera especificación formal reconocida (IETF, RFC 1866)                                   |
| HTML 3.2             | Ene 1997                 | W3C. Tablas, applets, alineación de imágenes                                                |
| HTML 4.0             | Dic 1997 (rev. abr 1998) | Mejor soporte de CSS, scripts, accesibilidad e internacionalización                         |
| HTML 4.01            | Dic 1999                 | Corrección y actualización de HTML 4                                                        |
| HTML5                | 2014 en adelante         | Semántica, audio, vídeo, formularios mejorados, API. HTML 5.2 fue recomendación W3C en 2017 |
| HTML Living Standard | Actual                   | Estándar vivo mantenido por WHATWG. No existe un "HTML 5.3" oficial                         |

Origen: Tim Berners-Lee desarrolló el sistema de hipertexto entre 1989 y 1990.

### HTML vs XHTML

XHTML es HTML reescrito con las reglas estrictas de XML. Un error de sintaxis, servido como XML, puede impedir que la página se muestre. HTML, en cambio, es permisivo y el navegador intenta corregir fallos.

| Regla                   | HTML                        | XHTML                                     |
| ----------------------- | --------------------------- | ----------------------------------------- |
| Mayúsculas en etiquetas | Se admiten                  | Todo en minúsculas                        |
| Comillas en atributos   | Opcionales en algunos casos | Siempre obligatorias                      |
| Atributos booleanos     | `checked`                   | `checked="checked"`                       |
| Elementos vacíos        | `<br>`                      | `<br />`                                  |
| Cierre de elementos     | Algunos se pueden omitir    | Todos cerrados y bien anidados            |
| Elemento raíz           | `<html>`                    | Un único `<html>`                         |
| Errores                 | El navegador corrige        | Documento mal formado: puede no mostrarse |

Qué pasa en la práctica: si el servidor envía el documento con el tipo MIME `application/xhtml+xml`, se procesa como XML (estricto). Si se envía como `text/html`, el navegador lo trata como HTML (permisivo). Hoy se escribe HTML del Living Standard y el término XHTML ya casi no se usa.

Ventajas de la sintaxis estricta: código más ordenado, detección rápida de errores y compatibilidad con herramientas XML (validadores, XSLT, analizadores).
