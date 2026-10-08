---
icon: html5
---

# Introducción

> Un **lenguaje de marcas** codifica un documento combinando el texto plano con **etiquetas, marcas o anotaciones**. Su objetivo es organizar la información para que la entiendan tanto personas como máquinas.

#### ¿Qué información aportan estas etiquetas?

1. **Estructura:** Identifican partes del documento (títulos, párrafos, enlaces, tablas).
2. **Semántica (Significado):** Indican qué representa el dato (ej. el autor, una fecha).
3. **Presentación:** Indican el aspecto visual (aunque la tendencia actual es separar esto del contenido).

### Reglas y Validación de Documentos

Para que un documento (especialmente en XML/SGML) esté correctamente estructurado, existen archivos que dictan las reglas:

| **Sistema**                          | **Descripción**                                                                                                        |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| **DTD** _(Document Type Definition)_ | Establece qué elementos, etiquetas y reglas de uso están permitidos en un documento. Es el sistema más clásico.        |
| **XSD** _(XML Schema)_               | Sistema más moderno. Hace lo mismo que la DTD, pero además permite definir **tipos de datos** (números, fechas, etc.). |
| **Excepción: HTML5**                 | Estándar del W3C. No necesita una DTD tradicional, se rige por su propia especificación oficial.                       |

### Clasificación de los Lenguajes de Marcas

Un documento puede mezclar varios tipos, pero se clasifican en 3 categorías principales:

1. **De Presentación:**
   * Indican _cómo debe verse_ el texto (formato, apariencia).
   * _Ejemplo:_ Etiquetas para poner negritas o cambiar el color.
2. **De Procedimientos:**
   * Contienen instrucciones.
   * Se deben interpretar por el programa exactamente en el **mismo orden** en que aparecen.
3. **Descriptivos o Semánticos (El estándar actual):**
   * Identifican **el significado** de las partes (esto es un título, esto es una dirección).
   * No determinan cómo deben mostrarse en pantalla.

### Evolución Histórica

La forma de estructurar documentos ha pasado por 3 grandes etapas:

| **Etapa**            | **Paradigma**         | **Características Principales**                                                                                                                             |
| -------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Inicial**       | _Dependencia_         | Códigos **propietarios** (atados a una máquina o programa). **No se podía separar** el contenido de su apariencia final.                                    |
| **2. Transición**    | _Interfaces Visuales_ | Nacen los botones y atajos (WYSIWYG). Las marcas se **ocultan al usuario**, pero el programa las sigue usando internamente para el formato.                 |
| **3. Normalización** | _Marcado Semántico_   | El marcado sirve para describir el **significado y estructura**, no la estética. **Separación total de capas**: Contenido (HTML/XML) vs Presentación (CSS). |

### Ámbitos de Aplicación y Ejemplos

Conoce los lenguajes más importantes según para qué se utilizan:

#### Documentación Electrónica

* **RTF (Microsoft):** Intercambio de texto enriquecido entre distintos procesadores.
* **TeX:** Documentos científicos y fórmulas matemáticas complejas.
* **Wikitexto:** Edición rápida en plataformas wiki (ej. MediaWiki).
* **DocBook (XML):** Separa estructura de presentación. Permite exportar a múltiples formatos (HTML, PDF, EPUB).

#### Tecnologías de Internet

* **HTML / XHTML:** Estructura de páginas web. **HTML5** es el estándar actual más extendido.
* **RSS:** Formato en XML para distribuir/sindicar novedades de blogs y medios digitales.

#### Lenguajes Especializados

* **MathML:** Representación de expresiones **matemáticas** entre aplicaciones.
* **VoiceXML:** Aplicaciones de reconocimiento y síntesis de **voz** (interacción hablada).
* **MusicXML:** Intercambio de **partituras** en programas de composición musical.

### Comparativa de Código

#### Antes (Marcado de Presentación)

Se centra solo en lo estético. Una máquina no sabe qué significan los datos.

```
<times 14><color verde><centrado> 
Ejemplo primitivo de presentación
</centrado></color></times 14>

```

#### Ahora (Marcado Semántico / XML)

Se centra en el **significado**. Cualquier sistema entiende quién envía la carta y cuándo.

```
<carta>
    <fecha>22/11/2006</fecha>
    <presentacion>Estimado cliente:</presentacion>
    <contenido>Bla bla bla...</contenido>
    <firma>Don José Gutiérrez</firma>
</carta>
```
