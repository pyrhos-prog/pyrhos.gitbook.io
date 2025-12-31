# requests

La librería `requests` es una de las más utilizadas en Python para realizar **peticiones HTTP** de forma sencilla.\
Se usa principalmente para interactuar con **APIs**, servicios web y recursos remotos.

#### Librería requests

**Funciones básicas**

* Permite realizar peticiones HTTP de forma simple
* Facilita el envío y recepción de datos desde servidores
* Maneja cabeceras, parámetros, cookies y autenticación

**Tipos de peticiones HTTP**

| **Tipo de petición** | **Función**                    | **Descripción**                   |
| -------------------- | ------------------------------ | --------------------------------- |
| GET                  | \<kbd>requests.get()\</kbd>    | Obtener información de un recurso |
| POST                 | \<kbd>requests.post()\</kbd>   | Enviar datos al servidor          |
| PUT                  | \<kbd>requests.put()\</kbd>    | Actualizar un recurso existente   |
| DELETE               | \<kbd>requests.delete()\</kbd> | Eliminar un recurso               |

**Uso básico de peticiones**

| **Acción**          | **Función**                  | **Detalles**                                      |
| ------------------- | ---------------------------- | ------------------------------------------------- |
| Petición simple     | \<kbd>requests.get()\</kbd>  | Solicitud básica sin parámetros                   |
| Envío de datos      | \<kbd>requests.post()\</kbd> | Envío de datos mediante body (formularios o JSON) |
| Envío de parámetros | \<kbd>params\</kbd>          | Parámetros incluidos en la URL                    |
| Envío de cabeceras  | \<kbd>headers\</kbd>         | Definir User-Agent, tokens, etc.                  |

**Ejemplos básicos**

Petición GET simple:

```python
import requests

response = requests.get("https://example.com")
print(response.status_code)
```

Petición GET con parámetros:

```python
params = {"q": "test", "page": 1}
response = requests.get("https://example.com/search", params=params)
```

**Envío de datos**

Enviar datos en un POST:

```python
data = {"user": "admin", "password": "1234"}
response = requests.post("https://example.com/login", data=data)
```

Enviar datos en formato JSON:

```python
json_data = {"name": "pyrhos", "role": "student"}
response = requests.post("https://example.com/api", json=json_data)
```

**Gestión de respuestas**

| **Elemento**       | **Uso**                            | **Descripción**                      |
| ------------------ | ---------------------------------- | ------------------------------------ |
| Código de estado   | \<kbd>response.status\_code\</kbd> | Estado HTTP de la respuesta          |
| Contenido en texto | \<kbd>response.text\</kbd>         | Respuesta en formato texto           |
| Contenido JSON     | \<kbd>response.json()\</kbd>       | Respuesta convertida a objeto Python |
| Cabeceras          | \<kbd>response.headers\</kbd>      | Cabeceras devueltas por el servidor  |

***

**Manejo de errores**

Comprobación básica del estado:

```python
if response.status_code == 200:
    print("Petición correcta")
else:
    print("Error en la petición")
```

**Casos de uso comunes**

* Consumo de APIs REST
* Automatización de peticiones web
* Pruebas de endpoints
* Scripts de scraping
