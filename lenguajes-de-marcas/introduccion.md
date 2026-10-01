---
icon: html5
---

# Introducción

> Los lenguajes de marcas permiten organizar y escribir la información para que pueda ser interpretada por personas y aplicaciones.

En el lenguaje de marcas, junto al texto se incorpora:

* Etiquetas
* Marcas
* Anotaciones\
  Para aportar información sobre su estructura, significado o presentación, Con ello podemos identificar elementos como titulos, parrafos, enlaces, tablas...

Nota

Un mismo documento puede guardar diferentes tipos de marcado

**Clasificación de los lenguajes de marcas:**

* _De presentación_: Indican el formato o apariencia del texto.
* _De procedimientos_: Contiene instrucciones que deben representarse en el mismo orden que aparecen en el documento.
* _Descriptivos o semánticos_: Identifican partes del documento sin determinar necesariamente como deben mostrarse.

## Evolución de los lenguajes de marcas

#### 1. Etapa inicial: Dependencia de plataforma

* **Códigos propietarios:** El marcado estaba atado a máquinas, programas o procesadores de texto específicos.
* **Falta de abstracción:** Resultaba imposible separar la estructura lógica del texto de su estilo o apariencia final.

#### 2. Transición: Interfaces visuales (WYSIWYG)

* **Inserción indirecta:** La edición manual de etiquetas se sustituyó por interfaces más accesibles (botones, menús, atajos de teclado).
* **Marcas ocultas:** Aunque el usuario dejaba de ver las etiquetas, las herramientas continuaban procesándolas internamente para definir el formato.

#### 3. Normalización: Marcado semántico

* **Cambio de paradigma:** El propósito del marcado evolucionó de dictar la _presentación_ a describir el _significado y la estructura_ del contenido.
* **Separación de capas:** Creación de estándares universales capaces de aislar por completo el contenido de su aspecto visual.

### Ejemplo

Código de marcas anterior a GML.

```markup
<times 14><color verde><centrado> Este texto es un ejemplo para mostrar la utilización primitiva de las marcas</centrado></color></times 14>
<color granate><times 10><cursiva>Para realiza este ejemplo se utilizan etiquetas de nuestra invención. </cursiva> 
Las partes importantes del texto pueden resaltarse usando la 
<negrita>negrita</negrita>, o el <subrayar>subrayado</subrayar></times 10></color>
```
