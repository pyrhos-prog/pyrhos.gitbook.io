# Metodos de solicitud

Los siguientes son algunos de los métodos de uso común:

| **Método** | **Descripción**                                                                                                                                       |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GET`      | Pide un recurso específico. Los datos adicionales se pueden pasar al servidor a través de cadenas de consulta en la URL (por ejemplo `?param=value`). |
| `POST`     | Envía datos al servidor. Se utiliza comúnmente cuando se envía información o para subir datos a un sitio web.                                         |
| `HEAD`     | Pide las cabeceras que serían devueltas si se hiciera una solicitud GET al servidor. No devuelve el cuerpo de la respuesta.                           |
| `PUT`      | Crea o reemplaza recursos en el servidor. Permitir este método sin los controles adecuados puede llevar a subir recursos maliciosos.                  |
| `DELETE`   | Elimina un recurso existente en el servidor web. Si no está correctamente asegurado, puede conducir a una Denegación de Servicio.                     |
| `OPTIONS`  | Devuelve información sobre el servidor, como los métodos aceptados por este.                                                                          |
| `PATCH`    | Aplica modificaciones parciales al recurso en la ubicación especificada.                                                                              |

{% embed url="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods" %}
