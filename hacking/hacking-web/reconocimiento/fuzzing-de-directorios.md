# Fuzzing de directorios

El fuzzing de directorios es una tecnica para descubrir rutas, archivos y recursos ocultos en un servidor web.

Se basa en probar automaticamente nombres de directorios y archivos mediante diccionarios.



### Objetivos del fuzzing de directorios

* Identificar paneles de administración ocultos
* Descubrir endpoints no documentados
* Encontrar archivos de configuración

### Funcionamiento básico

1. Se define una URL objetivo
2. Se sustituye una parte de la URL por una palabra variable
3. Se envían peticiones HTTP repetidas
4. Se analizan los códigos de respuesta

Ejemplo:

```
https://objetivo.com/FUZZ
```

### Códigos de respuesta relevantes

| 200       | Recurso existente  |
| --------- | ------------------ |
| 301 o 302 | Redirección        |
| 401       | Recurso protegido  |
| 403       | Acceso prohibido   |
| 404       | No existe          |
| 500       | Error del servidor |

### Limitaciones

* No descubre rutas dinámicas
* Puede ser bloqueado por WAF
* Depende en gran medida de la calidad de la wordlist
