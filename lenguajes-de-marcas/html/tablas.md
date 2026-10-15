---
icon: html5
---

# Tablas

## HTML: tablas

Las tablas muestran **datos tabulares**: horarios, inventarios, comparativas, listados. No se usan para maquetar la página; para distribuir el diseño existen Flexbox y Grid.

### Elementos

| Elemento   | Función                                             |
| ---------- | --------------------------------------------------- |
| `table`    | Contiene toda la tabla                              |
| `tr`       | Fila                                                |
| `th`       | Celda de encabezado                                 |
| `td`       | Celda de datos                                      |
| `caption`  | Título o descripción de la tabla                    |
| `thead`    | Agrupa las filas de cabecera                        |
| `tbody`    | Agrupa las filas principales de datos               |
| `tfoot`    | Agrupa las filas finales (totales, resúmenes)       |
| `colgroup` | Agrupa columnas para darles características comunes |

Jerarquía: `table` contiene `tr`; cada `tr` contiene `th` o `td`.

### Tabla mínima

```html
<table>
  <caption>Tabla de socios</caption>
  <tr>
    <th>Nombre</th>
    <th>Apellido</th>
    <th>Edad</th>
  </tr>
  <tr>
    <td>Juan</td>
    <td>Puertas</td>
    <td>54</td>
  </tr>
  <tr>
    <td>Eva</td>
    <td>Montes</td>
    <td>44</td>
  </tr>
</table>
```

### Tabla completa y accesible

Es la forma recomendada: título, secciones y `scope` en los encabezados.

```html
<table>
  <caption>Comparativa de ordenadores</caption>
  <thead>
    <tr>
      <th scope="col">Modelo</th>
      <th scope="col">RAM</th>
      <th scope="col">Precio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Portátil A</td>
      <td>16 GB</td>
      <td>899 EUR</td>
    </tr>
    <tr>
      <td>Portátil B</td>
      <td>32 GB</td>
      <td>1199 EUR</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td colspan="3">Precios con IVA incluido</td>
    </tr>
  </tfoot>
</table>
```

* `caption` describe la tabla.
* `th` + `scope="col"` relaciona cada encabezado con su columna (mejora la accesibilidad con lectores de pantalla).
* Datos normales en `td`. No intercambies `th` y `td`.

### Combinar celdas

| Atributo      | Efecto                      |
| ------------- | --------------------------- |
| `colspan="n"` | La celda ocupa `n` columnas |
| `rowspan="n"` | La celda ocupa `n` filas    |

```html
<tr>
  <th rowspan="2">Grupo</th>
  <td>Alumno 1</td>
</tr>
<tr>
  <td>Alumno 2</td>
</tr>
```

### Estilo

El borde, el espaciado y los colores se hacen con CSS, no con atributos HTML antiguos:

```css
table { border-collapse: collapse; width: 100%; }
th, td { border: 1px solid #999; padding: 8px; text-align: left; }
thead th { background-color: #eee; }
```

`border-collapse: collapse` fusiona los bordes de celdas contiguas en uno solo.

### Errores frecuentes

| Error                                                       | Corrección                                |
| ----------------------------------------------------------- | ----------------------------------------- |
| Usar `td` para encabezados                                  | Usar `th`                                 |
| Poner todos los datos en una sola fila separados con `<br>` | Una fila por registro, una celda por dato |
| Maquetar la página con tablas                               | CSS Flexbox o Grid                        |
| Tabla sin `caption` ni `scope`                              | Añadirlos: accesibilidad                  |
| Bordes con atributos HTML antiguos                          | CSS                                       |
