# Comando xargs

Sirve para convertir entradas estándares en argumentos de un comando, permitiendo ejecutar órdenes con listas largas o generadas dinámicamente.

## Opciones importantes de `xargs`

| Opción        | Significado                               |
| ------------- | ----------------------------------------- |
| `-n NUM`      | Número máximo de argumentos por ejecución |
| `-I MARCADOR` | Reemplaza MARCADOR por cada entrada       |
| `-0`          | Indica que la entrada usa terminador `\0` |
| `-p`          | Pregunta antes de ejecutar                |
| `-r`          | No ejecutar nada si no hay entrada        |

