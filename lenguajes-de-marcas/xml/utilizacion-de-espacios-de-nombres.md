---
icon: html5
---

# Utilización de Espacios de Nombres

### El problema: Ambigüedad de Etiquetas

Al ser XML un lenguaje extensible donde nosotros inventamos las etiquetas, surge un problema al integrar información de distintas fuentes.

Si combinamos dos documentos diferentes, podríamos encontrarnos con dos etiquetas idénticas que tienen significados distintos.&#x20;

Por ejemplo, si unimos un XML de una biblioteca y uno de la nobleza, la etiqueta `<titulo>` podría referirse al título de un libro o a un título nobiliario. El procesador XML no sabrá cómo diferenciarlos.

### La solución: Espacios de Nombres

Los espacios de nombres permiten indicar a qué vocabulario pertenecen los elementos, asociando un prefijo a cada conjunto de etiquetas.

#### ¿Cómo se construyen?

1. Se utiliza el atributo reservado `xmlns` (XML NameSpace).
2. Se define un prefijo corto (no puede contener espacios ni empezar por números).
3. Se le asigna un URI (Uniform Resource Identifier). Sirve como identificador único, no es necesario que sea una página web real.

Sintaxis de declaración: `<prefijo:etiqueta xmlns:prefijo="URI_identificador">`

### Ejemplo

Supongamos que tenemos datos de alumnos y de profesores en el mismo documento. Para evitar que la etiqueta `<nombre>` se confunda, creamos dos espacios de nombres: `alu` y `prof`.

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>

<!-- 1. Declaramos los Namespaces en el elemento raíz asociándolos a un URI -->
<instituto xmlns:alu="http://ejemplo.com/alumnos" 
           xmlns:prof="http://ejemplo.com/profesores">

    <alumnos>
        <!-- 2. Usamos el prefijo 'alu' para los nombres de alumnos -->
        <alu:nombre>Fernando Fernández González</alu:nombre>
        <alu:nombre>Isabel González Fernández</alu:nombre>
    </alumnos>

    <profesores>
        <!-- 3. Usamos el prefijo 'prof' para los nombres de profesores -->
        <prof:nombre>Pilar Ruiz Pérez</prof:nombre>
        <prof:nombre>Tomás Rodríguez Hernández</prof:nombre>
    </profesores> 
    
</instituto>
```

Al procesar este archivo, el sistema sabe perfectamente que `<alu:nombre>` y `<prof:nombre>` pertenecen a vocabularios distintos, resolviendo cualquier conflicto.
