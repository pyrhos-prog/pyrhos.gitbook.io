---
icon: html5
---

# XML

> **XML permite describir, organizar e intercambiar datos de forma independiente de su presentación.**&#x20;

#### XML es un **metalenguaje** caracterizado por:

* Permitir la creación de **etiquetas propias**.
* Incorporar atributos para añadir información a los elementos.
* Utilizar DTD o esquemas para definir etiquetas, atributos y restricciones.
* Mantener separadas la **estructura de los datos** y su presentación.

#### XML se relaciona con distintos estándares y tecnologías:

* **XSL (eXtensible Stylesheet Language):** permite presentar y transformar documentos XML. Incluye tecnologías como **XSLT** y XSL-FO.
* **XPath:** permite localizar y seleccionar elementos dentro de un documento XML.
* **XLink y XPointer:** permiten definir enlaces y referencias entre documentos o partes concretas de ellos.
* **XML Namespaces:** evitan conflictos entre etiquetas con el mismo nombre procedentes de vocabularios diferentes.
* **XML Schema (XSD):** permite definir con precisión la estructura, los tipos de datos y las restricciones de un documento XML. También pueden emplearse **DTD** o alternativas como Relax NG.

#### Ejemplo

```xml
<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE biblioteca">
<biblioteca>
    	<ejemplar tipo_ejem="libro" titulo="XML practico" editorial="Ediciones Eni">
        <tipo> <libro isbn="978-2-7460-4958-1" edicion="1" paginas="347"></libro> </tipo>
        <autor nombre="Sebastien Lecomte"></autor>
        <autor nombre="Thierry Boulanger"></autor>
        <autor nombre="Ángel Belinchon Calleja" funcion="traductor"></autor>
        <prestado lector="Pepito Grillo">
            <fecha_pres dia="13" mes="mar" año="2009"></fecha_pres>
            <fecha_devol dia="21" mes="jun" año="2009"></fecha_devol>
        </prestado> 
    </ejemplar>
    <ejemplar tipo_ejem="revista" titulo="Todo Linux 101. Virtualización en GNU/Linux" editorial="Studio Press">
        <tipo>
            <revista>
                <fecha_publicacion mes="abr" año="2009"></fecha_publicacion> 
            </revista> 
        </tipo>
        <autor nombre="Varios"></autor>
        <prestado lector="Pedro Picapiedra">
            <fecha_pres dia="12" mes="ene" año="2010"></fecha_pres>
        </prestado> 
    </ejemplar>
</biblioteca>
```

#### HTML vs XML&#x20;

| Es un **metalenguaje derivado de SGML**.                                                  | Se originó como una aplicación de SGML y actualmente se define mediante el estándar **HTML Living Standard**. |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Permite definir etiquetas adaptadas a cada tipo de información.                           | Utiliza un conjunto de etiquetas previamente establecido.                                                     |
| Se orienta a **describir, almacenar e intercambiar datos**.                               | Se orienta a estructurar el contenido de las páginas web.                                                     |
| Las etiquetas indican el significado de los datos, como **\<cliente>** o **\<pedido>**.   | Las etiquetas identifican elementos como títulos, párrafos, imágenes o enlaces.                               |
| Exige una sintaxis estricta: las etiquetas deben cerrarse y estar correctamente anidadas. | Los navegadores pueden corregir algunos errores de sintaxis al interpretar el documento.                      |
| Puede utilizar tecnologías como **XPath, XSLT, XLink o XSD**.                             | Utiliza enlaces sencillos mediante elementos como **\<a>** y se combina con CSS y JavaScript.                 |
| Puede validarse mediante DTD o esquemas XML.                                              | Se valida según las reglas definidas por el estándar HTML.                                                    |
| No define por sí mismo cómo se muestran los datos.                                        | El navegador interpreta el documento y representa su contenido en pantalla.                                   |
