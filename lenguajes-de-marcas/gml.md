---
icon: html5
---

# GML

> **GML** (_Generalized Markup Language_, Lenguaje de Marcado Generalizado) es un lenguaje concebido para describir la **estructura lógica** y los **elementos semánticos** de un documento (títulos, párrafos, listas, tablas) de forma totalmente independiente de su aspecto visual final, del hardware y del software utilizado para procesarlo.

* **Año de creación:** 1969.
* **Entorno:** Desarrollado dentro de **IBM**.
* **Autores:** Charles **G**oldfarb, Edward **M**osher y Raymond **L**orie.

| Característica                      | Descripción                                                                                                                                                                          |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Independencia de plataforma**     | Al basarse en texto estructurado, los documentos podían procesarse en mainframes, terminales o archivarse sin dependencia de formatos propietarios binarios.                         |
| **Marcado declarativo / semántico** | No especifica tipografía ni márgenes; indica roles estructurales (capítulo, aviso, cita).                                                                                            |
| **Procesamiento por perfiles**      | Un mismo archivo fuente en GML podía derivar en una versión impresa en alta calidad o en un texto formateado para pantalla mediante diferentes scripts de formateo (como SCRIPT/VS). |
| **Sintaxis basada en etiquetas**    | Utilizaba etiquetas delimitadas habitualmente por dos puntos (`:tag.` o `:etag.`).                                                                                                   |

#### Ejemplo de sintaxis GML:

```
:h1.Introducción a los Sistemas Operativos
:p.Un sistema operativo gestiona los recursos del hardware.
:ol.
:li.Gestión de memoria
:li.Planificación de procesos
:eol.
```



GML es el **antepasado directo** de los lenguajes de marcado estructurado modernos:

```
       [1969] GML (IBM)
              │
              ▼
       [1986] SGML (ISO 8879)
              ├──► [1991] HTML (Aplicación / DTD de SGML)
              └──► [1998] XML (Subconjunto estricto y simplificado de SGML)
```

1. **SGML (Standard Generalized Markup Language, 1986 - ISO 8879):**
   * Charles Goldfarb continuó el desarrollo de GML en comités de normalización, dando lugar al estándar formal internacional SGML.
   * Introdujo de forma rigurosa el concepto de **DTD** (_Document Type Definition_).
2. **HTML (HyperText Markup Language):**
   * Creado por Tim Berners-Lee a partir de las ideas de SGML para compartir hipertexto científico en la web.
3. **XML (eXtensible Markup Language):**
   * Diseñado por el W3C para llevar la flexibilidad de SGML a la web, eliminando su excesiva complejidad sintáctica.
