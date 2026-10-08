---
icon: html5
---

# Documento XML bien formado

> En XML, un documento "bien formado" es aquel que cumple estrictamente con todas las reglas de sintaxis del lenguaje. Si un documento XML no está bien formado, el procesador (o navegador) lanzará un error y detendrá la lectura de los datos.

**Para que un documento XML esté bien formado, debe cumplir las siguientes reglas:**

### Las reglas para un documento bien formado

1. **ELEMENTO RAÍZ ÚNICO** Todo el contenido del documento debe estar encerrado dentro de un único elemento padre. No puede haber dos elementos en el nivel superior.
2. **SENSIBILIDAD A MAYÚSCULAS Y MINÚSCULAS** XML es "case-sensitive". La etiqueta de apertura y la de cierre deben estar escritas exactamente igual.
   1. \[CORRECTO] `<Nombre>Juan</Nombre>`
   2. \[ERROR] `<Nombre>Juan</nombre>`
3. **CIERRE OBLIGATORIO DE ETIQUETAS** Toda etiqueta que se abre, debe cerrarse. A diferencia de versiones antiguas de HTML (donde etiquetas como `<br>` no se cerraban), en XML es obligatorio. Los elementos vacíos deben cerrarse sobre sí mismos: `<br/>`.
4. **ANIDAMIENTO PERFECTO** Los elementos deben cerrarse en el orden inverso al que se abrieron. Las etiquetas no pueden cruzarse.
   1. \[CORRECTO] `<negrita><cursiva>Texto</cursiva></negrita>`
   2. \[ERROR] `<negrita><cursiva>Texto</negrita></cursiva>`
5. **VALORES DE ATRIBUTOS ENTRE COMILLAS** Los valores asignados a cualquier atributo deben ir siempre encerrados entre comillas dobles o simples.
   1. \[CORRECTO] `<libro categoria="ficcion">`
   2. \[ERROR] `<libro categoria=ficcion>`
6. **CARACTERES RESERVADOS** No se pueden utilizar caracteres que pertenezcan a la sintaxis del lenguaje dentro de los datos (como `<` o `&`). En su lugar, se deben usar las entidades correspondientes (`&lt;` y `&amp;`).
