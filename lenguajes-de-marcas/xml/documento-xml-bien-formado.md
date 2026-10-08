---
icon: html5
---

# Documento XML bien formado

> En XML, un documento "bien formado" es aquel que cumple estrictamente con todas las reglas de sintaxis del lenguaje. Si un documento XML no está bien formado, el procesador (o navegador) lanzará un error y detendrá la lectura de los datos.

**Para que un documento XML esté bien formado, debe cumplir las siguientes reglas:**

Si un documento XML incumple una sola de estas reglas, el procesador (parser) devolverá un error fatal y detendrá la lectura.

1. **Elemento raíz único:** Todo el documento debe estar contenido dentro de una única etiqueta principal.
2. **Cierre de etiquetas:** Toda etiqueta que se abre, debe cerrarse obligatoriamente. Si está vacía, debe cerrarse sobre sí misma (ej. `<etiqueta/>`).
3. **Anidamiento correcto:** Las etiquetas deben cerrarse en el orden inverso al que se abrieron. No pueden cruzarse.
   * \[CORRECTO] `<a> <b> </b> </a>`
   * \[ERROR] `<a> <b> </a> </b>`
4. **Sensibilidad a mayúsculas/minúsculas:** XML es case-sensitive. `<Nombre>` y `<nombre>` son consideradas etiquetas completamente distintas.
5. **Atributos entre comillas:** Los valores de cualquier atributo deben ir siempre entre comillas simples o dobles.
   * \[CORRECTO] `<usuario id="15">`
   * \[ERROR] `<usuario id=15>`
6. **Caracteres especiales escapados:** No se pueden usar libremente símbolos como `<` o `&` en el texto, ya que el procesador pensará que empieza una etiqueta. Deben usarse entidades (ej. `&lt;` para `<`).

### Concepto Clave: Bien Formado vs. Válido

Este es uno de los conceptos más importantes en XML. No es lo mismo que un documento esté bien escrito (Bien Formado) a que cumpla las reglas de negocio de una empresa (Válido).

| **Bien Formado** | Cumple la sintaxis básica de XML (tiene raíz, etiquetas cerradas, etc.). El código no tiene errores de escritura.        | **No.** | No es necesario.             |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------ | ------- | ---------------------------- |
| **Válido**       | **Además de estar Bien Formado**, cumple estrictamente con la estructura, orden y datos dictados por un esquema externo. | **Sí.** | Sí, requiere la declaración. |

#### Ejemplo: Una Factura

Imagina que tienes que hacer una factura para cobrar un trabajo.

* **El documento "Bien Formado":** Escribes tu factura en una hoja en blanco. Pones tu nombre, el servicio realizado y el precio final. Está escrita sin faltas de ortografía, con buena letra y se lee perfectamente. A nivel técnico de XML, tu documento está "Bien Formado".
* **El documento "Válido":** Le entregas la factura al contable de la empresa, y te la devuelve rechazada. Te dice: _"Le falta la fecha, el número de factura y el NIF de la empresa"_. El contable está actuando como el archivo **DTD** (las reglas externas).

Para que tu factura sea **Válida**, no solo tiene que estar bien escrita (Bien Formada), sino que tiene que rellenarse utilizando el "molde" estricto que exige el contable (el DTD o DOCTYPE). Si el molde dice que la etiqueta `<nif>` es obligatoria y tú no la pones, el documento dará un error de validación, aunque sintácticamente estuviera perfecto.

#### Conclusión práctica

* **Sin DOCTYPE:** Tienes un folio en blanco. Puedes inventarte las etiquetas que quieras. Es rápido, pero corres el riesgo de olvidar datos importantes y que el programa que lo lea falle.
* **Con DOCTYPE:** Tienes un formulario estricto. Estás atando el XML a un "contrato". Si te saltas una regla de ese contrato, el documento dará error por ser Inválido.
