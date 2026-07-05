---
icon: spider-web
---

# FFuF

> FFuF (Fuzz Faster U Fool) es un fuzzer web escrito en Go, orientado a velocidad. Permite enumerar directorios, archivos, extensiones, subdominios, vhosts y parámetros mediante sustitución de palabras clave en peticiones HTTP.

## Instalación y opciones

#### Instalación

| Método           | Comando                                     |
| ---------------- | ------------------------------------------- |
| Repositorios apt | `apt install ffuf -y`                       |
| Go install       | `go install github.com/ffuf/ffuf/v2@latest` |
| GitHub (release) | `github.com/ffuf/ffuf`                      |
| PwnBox           | Preinstalado                                |

#### Sintaxis básica

La palabra clave `FUZZ` marca el punto de inserción de cada línea de la wordlist. Puede colocarse en la URL, cabeceras, cuerpo POST o incluso asociarse a varias wordlists con nombres distintos (`-w wordlist:KEYWORD`).

```bash
ffuf -w /ruta/wordlist.txt -u http://IP:PORT/FUZZ
```

#### Opciones principales

| Flag                                  | Descripción                                             |
| ------------------------------------- | ------------------------------------------------------- |
| `-w`                                  | Ruta de la wordlist, opcionalmente `ruta:PALABRA_CLAVE` |
| `-u`                                  | URL objetivo                                            |
| `-X`                                  | Método HTTP (por defecto GET)                           |
| `-H`                                  | Cabecera `"Nombre: Valor"` (se puede repetir)           |
| `-b`                                  | Cookies `"NOMBRE1=VAL1; NOMBRE2=VAL2"`                  |
| `-d`                                  | Datos POST                                              |
| `-e`                                  | Extensiones a probar, ej. `.php,.html,.txt,.bak,.js`    |
| `-t`                                  | Número de hilos (por defecto 40)                        |
| `-mc` / `-ms` / `-ml` / `-mw` / `-mr` | Coincidir por código, tamaño, líneas, palabras, regex   |
| `-fc` / `-fs` / `-fl` / `-fw` / `-fr` | Filtrar por código, tamaño, líneas, palabras, regex     |
| `-recursion` / `-recursion-depth`     | Escaneo recursivo y su profundidad máxima               |
| `-o`                                  | Guardar salida en fichero                               |
| `-v`                                  | Salida detallada (verbose)                              |
| `-c`                                  | Salida coloreada                                        |

## Descubrimiento de contenido

### Directorios

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ \
     -u http://IP:PORT/FUZZ
```

Resultado esperado: códigos 200-399 (por defecto ffuf también incluye 401/403 como matcher), indicando existencia del recurso aunque no siempre acceso completo.

Precaución: subir `-t` (hilos) acelera el escaneo pero incrementa el riesgo de provocar una denegación de servicio (DoS) en el objetivo, o de saturar la propia conexión si se lanza desde red local o VPN limitada. En entornos de cliente real (EVOLK), mantener el valor por defecto o negociar el ritmo de escaneo antes de aumentar hilos.

### Archivos

| Extensión | Contenido típico                        |
| --------- | --------------------------------------- |
| `.php`    | Código del lado del servidor            |
| `.html`   | Estructura y contenido de páginas       |
| `.txt`    | Texto plano, notas o logs               |
| `.bak`    | Copias de seguridad de archivos previos |
| `.js`     | Lógica de cliente / JavaScript          |

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
     -u http://IP:PORT/directorio_encontrado/FUZZ \
     -e .php,.html,.txt,.bak,.js -v
```

`-v` muestra la URL completa y la palabra usada (`FUZZ: nombre`) para cada hallazgo.

### Identificacion de extensión

Si no se conoce la tecnología del sitio, deducirla por la cabecera del servidor (Apache → probable PHP, IIS → probable ASP/ASPX) es poco fiable. Es más práctico hacer fuzzing de la propia extensión sobre un archivo que casi siempre existe, como `index`:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ \
     -u http://IP:PORT/directorio/indexFUZZ
```

Notas:

* La wordlist de extensiones ya incluye el punto (`.php`, `.html`...), no se añade manualmente tras `index`.
* Para fuzzing simultáneo de nombre y extensión: dos wordlists con palabras clave distintas, combinadas como `FUZZ_1.FUZZ_2`.
* Solo un código 200 (frente a 403 u otros) confirma que esa extensión es la que realmente sirve contenido.

### Páginas con la extensión confirmada

Una vez conocida la extensión real, se hace fuzzing de nombres de archivo con esa extensión fija, reutilizando la wordlist de directorios:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ \
     -u http://IP:PORT/directorio/FUZZ.php
```

Interpretación: dos hallazgos con código 200 no son necesariamente iguales — tamaño de respuesta 0 indica página vacía, mientras que un tamaño mayor indica contenido real que merece revisión manual.

## Fuzzing recursivo

Hacer fuzzing manual directorio por directorio no escala cuando hay muchos subdirectorios anidados. El fuzzing recursivo automatiza esto: al encontrar un nuevo directorio, lanza automáticamente un nuevo job de fuzzing dentro de él.

| Flag               | Descripción                                                       |
| ------------------ | ----------------------------------------------------------------- |
| `-recursion`       | Activa el escaneo recursivo (requiere que `-u` termine en `FUZZ`) |
| `-recursion-depth` | Límite de profundidad (ej. `1` = solo subdirectorios directos)    |

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ \
     -u http://IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

Consideraciones:

* `-e .php` sigue siendo válido en modo recursivo porque normalmente la extensión es la misma en todo el sitio.
* `-v` es casi obligatorio: sin URLs completas es difícil saber bajo qué directorio cae cada archivo hallado.
* Coste alto: la wordlist se duplica (con y sin extensión) y las peticiones pueden multiplicarse por 5-6x respecto a un escaneo plano.
* Estrategia recomendada: escaneo plano rápido primero para identificar directorios de interés, y solo entonces recursión limitada (`-recursion-depth 1` o `2`) sobre esos directorios concretos.

## Descubrimiento de dominios

### Subdominios

Comprueba si existe un registro DNS público para cada palabra de la wordlist, colocando `FUZZ` en la posición del subdominio:

```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ \
     -u https://FUZZ.inlanefreight.com/
```

Un `Errors: 4997` con cero resultados en un dominio como `academy.htb` no significa que no existan subdominios — significa que no tienen registro DNS público. `/etc/hosts` solo resuelve el dominio principal, no los subdominios que ffuf va probando, por lo que la consulta cae en el DNS público, que no los conoce.

### Vhosts

Un vhost es, en esencia, un "subdominio" servido desde la misma IP: un único servidor puede alojar varios sitios distintos según la cabecera `Host`, con o sin registro DNS público. Para detectarlos sin tocar `/etc/hosts` por cada palabra, se hace fuzzing directamente sobre la cabecera `Host`:

```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ \
     -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

Aquí casi todas las palabras devuelven `200 OK`, porque en realidad se sigue visitando la misma URL base cambiando solo la cabecera. Eso es esperable; el vhost real se distingue por un tamaño de respuesta distinto al resto, no por el código de estado.

## Filtrado de resultados

Por defecto ffuf solo descarta 404; el resto de códigos (incluidos muchos 200 "falsos") se mantienen. Cuando eso genera ruido (como en el ejemplo de vhosts anterior, donde todo responde 200 con tamaño 900), hay que filtrar por otro criterio.

| Situación                                                                | Filtro recomendado    |
| ------------------------------------------------------------------------ | --------------------- |
| Tamaño de respuesta uniforme conocido (soft-404 o respuesta por defecto) | `-fs <tamaño>`        |
| Código de estado uniforme no útil                                        | `-fc <código>`        |
| Número de palabras/líneas uniforme                                       | `-fw <n>` / `-fl <n>` |

```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ \
     -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```

Con el filtro aplicado, el ruido desaparece y queda un único resultado real (ej. `admin`, tamaño 0). Se verifica visitando `https://admin.academy.htb:PORT/` (tras añadirlo a `/etc/hosts`) y comprobando que responde distinto al dominio principal — página vacía en vez de contenido, rutas conocidas devolviendo 404, etc.

Método general para encontrar el valor de filtro: probar una entrada que sabemos que no existe, anotar su código/tamaño/palabras, y filtrar ese patrón para el resto del escaneo.

## Fuzzing de parámetros y valores

### Parámetros GET

Los parámetros GET se pasan en la URL tras `?` (`?param1=key`). Basta con sustituir el nombre del parámetro por `FUZZ`:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ \
     -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

Filtrar por el tamaño de respuesta por defecto (`-fs`) es necesario porque, igual que con vhosts, la mayoría de intentos devuelven una respuesta uniforme (parámetro no reconocido).

### Parámetros POST

Los datos POST no van en la URL sino en el campo `data` de la petición. Se usa `-d` para el cuerpo y `-X POST` para el método:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ \
     -u http://admin.academy.htb:PORT/admin/admin.php \
     -X POST -d 'FUZZ=key' \
     -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

En PHP, el `Content-Type` de datos POST normalmente solo acepta `application/x-www-form-urlencoded`; conviene fijarlo explícitamente con `-H`.

Un parámetro encontrado por fuzzing puede confirmarse después con una petición manual (`curl -X POST -d 'param=valor' ...`) para leer la respuesta completa.

Nota: el fuzzing de parámetros puede exponer parámetros no publicados y de acceso público que suelen estar peor probados y menos asegurados — candidatos naturales para las vulnerabilidades web habituales (IDOR, inyección, etc.).

## Fuzzing con wordlist personalizada

Encontrar el nombre del parámetro correcto no basta si también hace falta el valor correcto. No siempre existe una wordlist prehecha para eso — depende de qué tipo de dato espera el parámetro (usuario, id numérico, token...).

Para un parámetro tipo `id` que se sospecha numérico y posiblemente secuencial, se puede generar una wordlist propia de números:

```bash
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```

Y aplicarla sobre el valor del parámetro ya confirmado:

```bash
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php \
     -X POST -d 'id=FUZZ' \
     -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

El flujo completo de fuzzing de parámetros es: **nombre del parámetro (GET/POST) → filtrar ruido por tamaño → valor del parámetro con wordlist a medida → confirmar con curl.**
