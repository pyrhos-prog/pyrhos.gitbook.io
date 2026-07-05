---
icon: spider-web
---

# FFuF

> FFuF (Fuzz Faster U Fool) es un fuzzer web escrito en Go, orientado a velocidad. Permite enumerar directorios, archivos, parámetros, subdominios y credenciales mediante sustitución de palabras clave en peticiones HTTP.

### Instalación

| Método           | Comando                                     |
| ---------------- | ------------------------------------------- |
| Repositorios apt | `apt install ffuf -y`                       |
| Go install       | `go install github.com/ffuf/ffuf/v2@latest` |
| GitHub (release) | `github.com/ffuf/ffuf`                      |
| PwnBox           | Preinstalado                                |

### Sintaxis básica

La palabra clave `FUZZ` marca el punto de inserción de cada línea de la wordlist. Puede colocarse en la URL, cabeceras, cuerpo POST o incluso asociarse a varias wordlists con nombres distintos (`-w wordlist:KEYWORD`).

```bash
ffuf -w /ruta/wordlist.txt -u http://IP:PORT/FUZZ
```

### Opciones principales

| Flag               | Descripción                                             |
| ------------------ | ------------------------------------------------------- |
| `-w`               | Ruta de la wordlist, opcionalmente `ruta:PALABRA_CLAVE` |
| `-u`               | URL objetivo                                            |
| `-X`               | Método HTTP (por defecto GET)                           |
| `-H`               | Cabecera `"Nombre: Valor"` (se puede repetir)           |
| `-b`               | Cookies `"NOMBRE1=VAL1; NOMBRE2=VAL2"`                  |
| `-d`               | Datos POST                                              |
| `-e`               | Extensiones a probar, ej. `.php,.html,.txt,.bak,.js`    |
| `-t`               | Número de hilos (por defecto 40)                        |
| `-mc`              | Códigos de estado a incluir, o `all`                    |
| `-ms`              | Tamaño de respuesta a incluir                           |
| `-fc`              | Filtrar por código de estado                            |
| `-fs`              | Filtrar por tamaño de respuesta                         |
| `-fw`              | Filtrar por número de palabras                          |
| `-recursion`       | Escaneo recursivo (requiere que `-u` termine en FUZZ)   |
| `-recursion-depth` | Profundidad máxima de recursión                         |
| `-o`               | Guardar salida en fichero                               |
| `-v`               | Salida detallada (verbose)                              |
| `-c`               | Salida coloreada                                        |

### Fuzzing de directorios

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ \
     -u http://IP:PORT/FUZZ
```

Resultado esperado: códigos 200-399 (por defecto ffuf también incluye 401/403 como matcher), indicando existencia del recurso aunque no siempre acceso completo.

Precaución: subir `-t` (hilos) acelera el escaneo pero incrementa el riesgo de:

* Provocar una denegación de servicio (DoS) en el objetivo.
* Saturar la propia conexión si se lanza desde red local o VPN limitada.

En entornos de cliente real (EVOLK), mantener el valor por defecto o negociar el ritmo de escaneo antes de aumentar hilos.

### Fuzzing de archivos

Una vez identificado un directorio, se profundiza buscando archivos con extensiones específicas dentro de él.

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

`-v` es útil aquí porque muestra la URL completa y la palabra usada (`FUZZ: nombre`) para cada hallazgo, en vez de solo el resumen de línea.

### Fuzzing de extensiones

Antes de buscar páginas concretas conviene saber qué extensión usa el sitio (`.php`, `.aspx`, `.html`...). Deducirlo por la cabecera del servidor (Apache → probable PHP, IIS → probable ASP/ASPX) es poco fiable; es más práctico hacer fuzzing de la propia extensión sobre un archivo que casi siempre existe, como `index`.

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ \
     -u http://IP:PORT/directorio/indexFUZZ
```

Notas:

* La wordlist de extensiones ya incluye el punto (`.php`, `.html`...), por lo que no se añade manualmente tras `index`.
* Si se quiere fuzzing simultáneo de nombre y extensión, se usan dos wordlists con palabras clave distintas y se combinan como `FUZZ_1.FUZZ_2`.
* Solo un código 200 (frente a 403 u otros) confirma que esa extensión es la que realmente sirve contenido.

### Fuzzing de páginas

Una vez conocida la extensión real, se hace fuzzing de nombres de archivo con esa extensión fija, reutilizando la misma wordlist de directorios.

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ \
     -u http://IP:PORT/directorio/FUZZ.php
```

Interpretación de resultados: dos hallazgos con código 200 no son necesariamente iguales — un tamaño de respuesta 0 indica página vacía, mientras que un tamaño mayor indica contenido real que merece revisión manual.

### Fuzzing recursivo

Hacer fuzzing manual directorio por directorio, y dentro de cada uno archivo por archivo, no escala cuando hay muchos subdirectorios anidados (`/login/user/content/uploads/...`). El fuzzing recursivo automatiza ese proceso: al encontrar un nuevo directorio, lanza automáticamente un nuevo job de fuzzing dentro de él.

| Flag               | Descripción                                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------------- |
| `-recursion`       | Activa el escaneo recursivo (requiere que `-u` termine en `FUZZ`)                                  |
| `-recursion-depth` | Límite de profundidad de recursión (ej. `1` = solo subdirectorios directos, no sub-subdirectorios) |

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ \
     -u http://IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

Consideraciones:

* `-e .php` sigue siendo válido en modo recursivo porque normalmente la extensión es la misma en todo el sitio.
* `-v` es casi obligatorio aquí: sin URLs completas es difícil saber bajo qué directorio cae cada archivo hallado.
* El coste es alto: la wordlist se duplica (con y sin extensión) y el número de peticiones puede multiplicarse por 5-6x respecto a un escaneo plano, con la consiguiente subida de tiempo y ruido en el objetivo.
* Estrategia recomendada: primero un escaneo plano rápido para identificar directorios de interés, y solo entonces lanzar recursión limitada (`-recursion-depth 1` o `2`) sobre esos directorios concretos, en vez de recursión sin límite desde la raíz.

### Filtrado de falsos positivos

Muchos servidores devuelven 200 para cualquier ruta (soft-404). Antes de confiar en los resultados:

1. Probar una ruta aleatoria que sabemos no existe (`/asdkjhasdkjh123`).
2. Anotar código, tamaño y palabras de esa respuesta "vacía".
3. Filtrar ese patrón con `-fc` (código) o `-fs` (tamaño) para limpiar el resto del escaneo.

```bash
ffuf -w wordlist.txt -u http://IP:PORT/FUZZ -fs 42
```

### Fuzzing de vhosts

```bash
ffuf -w subdominios.txt -u http://IP:PORT/ \
     -H "Host: FUZZ.dominio.com" -fs <tamaño_respuesta_default>
```

Útil cuando un mismo servidor aloja varios sitios distintos según la cabecera `Host` (virtual hosting), habitual en reconocimiento de infraestructura de clientes con múltiples dominios.
