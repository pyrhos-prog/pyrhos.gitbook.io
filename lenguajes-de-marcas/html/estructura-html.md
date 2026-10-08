---
icon: html5
---

# Estructura HTML

> **La estructura básica de HTML permite que el navegador identifique correctamente la página, sus metadatos y el contenido visible.**

### Declaración del tipo de documento

Todo documento HTML moderno debe comenzar con la siguiente línea:

```html
<!DOCTYPE html>
```

* Indica al navegador que interprete la página según el estándar HTML actual (HTML5).
* Ya no hace referencia a una DTD (Document Type Definition) ni a una versión concreta.

{% hint style="info" %}
HTML 4.01 (Obsoleto) Antiguamente existían tres declaraciones, que ya no deben usarse en páginas nuevas:\
\- Strict: No permitía elementos de presentación obsoletos.\
\- Transitional: Admitía elementos antiguos (para facilitar la adaptación).\
\- Frameset: Se usaba en documentos organizados mediante marcos.
{% endhint %}

### El Documento HTML: Elemento raíz

Todo el contenido se engloba dentro de la etiqueta `<html>`. Lo habitual y recomendado es incluir el atributo `lang` para definir el idioma de la página:

HTML

```html
<html lang="es">
```

Dentro del elemento raíz, el documento se divide en dos partes principales:

#### Cabecera (`<head>`)

Contiene los metadatos (información que _no_ forma parte del contenido principal visible).

* `<title>`: Define el nombre que aparecerá en la pestaña del navegador.
* Otros metadatos comunes: Codificación de caracteres (charset), enlaces a hojas de estilo (CSS), y adaptación a dispositivos móviles (viewport).

#### Cuerpo (`<body>`)

Contiene la información visible de la página web.

* Aquí van los títulos, párrafos, imágenes, enlaces, tablas, formularios, etc.

### Estructura Básica

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

{% hint style="warning" %}
Las etiquetas `<frameset>` y `<frame>`  están completamente obsoletas. Actualmente, la distribución del contenido en la pantalla se realiza utilizando HTML semántico y hojas de estilo CSS.
{% endhint %}
