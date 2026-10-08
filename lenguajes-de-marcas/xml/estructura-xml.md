---
icon: html5
---

# Estructura XML

> Un documento XML tiene una estructura de "árbol" estricta y se divide fundamentalmente en dos grandes bloques: el Prólogo y el Ejemplar (el contenido en sí).

## 1. El prólogo

Se ubica siempre en la primera línea del archivoy su uso opcional pero es muy recomendado ponerlo. Su función es dar instrucciones al analizador o navegador sobre cómo debe interpretar el documento.&#x20;

**Dentro del prólogo hay 2 valores:**

### La Declaración XML

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
```

* version="1.0" - Indica la versión del estándar XML utilizada.
* encoding="..." - Define la codificación de caracteres. "UTF-8" es la recomendada (admite la ñ y tildes). Otra común es "ISO-8859-1" (Europa occidental).
* standalone="..." - Determina si el documento es autónomo ("yes") o si requiere archivos externos para ser interpretado ("no").

### La Declaración del Tipo de Documento

Se utiliza para vincular el documento XML con el archivo que dicta sus reglas (una DTD).

```
<!DOCTYPE Nombre_Raiz SYSTEM "archivo.dtd">
```

## 2. El ejemplar y los elementos

Esta es la parte que contiene la información. Todo el documento debe estar contenido obligatoriamente dentro de UNA única etiqueta principal llamada Elemento Raíz.

### Componentes de un Elemento

1. **Etiquetas de Apertura y Cierre:** Delimitan el inicio y el fin del dato.
2. **Contenido (Datos de caracteres):** La información real entre las etiquetas.
3. **Atributos:** Información adicional sobre el elemento. Se colocan dentro de la etiqueta de apertura y su valor siempre va entre comillas.

```
<usuario id="001">                         <!-- Etiqueta APERTURA con ATRIBUTO -->
    <nombre>Laura</nombre>                 <!-- ELEMENTO ANIDADO con su DATO -->
    <correo>laura@ejemplo.com</correo>     <!-- ELEMENTO ANIDADO con su DATO -->
</usuario>                                 <!-- Etiqueta de CIERRE del Elemento Raíz -->
```

### Comentarios y Entidades

* **Comentarios:** Sirven para documentar el código. No son procesados por las aplicaciones. Sintaxis: `<!-- Texto del comentario -->` (Nota: No pueden ir dentro de una etiqueta ni contener la secuencia de dos guiones seguidos `--`).
* **Entidades:** Códigos especiales para representar caracteres reservados y evitar que el procesador se confunda. Ejemplo: Usar `&lt;` para el símbolo menor que (`<`) o `&amp;` para el ampersand (`&`).
