---
icon: globe-www
---

# Fundamentos

> El ataque de fuerza bruta (brute forcing) es un método de ensayo y error para descifrar contraseñas, credenciales o claves de cifrado, probando sistemáticamente combinaciones hasta encontrar la correcta.

### Factores de éxito

| Factor                             | Efecto                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------------------ |
| Complejidad de la contraseña/clave | A mayor longitud y variedad de caracteres, exponencialmente más difícil de descifrar |
| Potencia de cómputo del atacante   | Hardware especializado permite probar miles de millones de combinaciones/segundo     |
| Medidas de seguridad del objetivo  | Account lockout, CAPTCHA y similares ralentizan o frustran el ataque                 |

### Proceso

1. Inicio del ataque (normalmente con herramienta especializada).
2. Generación de una combinación candidata según parámetros definidos (charset, longitud).
3. Envío de la combinación contra el objetivo (login, archivo cifrado, etc.).
4. Comprobación de éxito: coincide → acceso concedido; no coincide → se repite el proceso.

### Matemática de las combinaciones

```
Combinaciones posibles = (Tamaño del charset) ^ (Longitud de la contraseña)
```

| Longitud | Charset                                      | Combinaciones             |
| -------- | -------------------------------------------- | ------------------------- |
| 6        | Minúsculas (a-z)                             | 26^6 = 308,915,776        |
| 8        | Minúsculas (a-z)                             | 26^8 = 208,827,064,576    |
| 8        | Minúsculas + mayúsculas                      | 52^8 = 53,459,728,531,456 |
| 12       | Minúsculas + mayúsculas + números + símbolos | 94^12 ≈ 4.76 × 10^23      |

Un pequeño aumento de longitud o charset expande el espacio de búsqueda exponencialmente. El tiempo real de crackeo depende también de la potencia de cómputo disponible, no solo del tamaño del espacio:

| Hardware         | Velocidad             | 8 car. (letras+dígitos) | 12 car. (ASCII completo) |
| ---------------- | --------------------- | ----------------------- | ------------------------ |
| Equipo básico    | \~1M intentos/s       | \~6.92 años             | Impracticable            |
| Supercomputadora | \~1 billón intentos/s | Segundos/minutos        | \~15,000 años            |

Conclusión práctica: incluso con hardware masivo, la longitud sigue ganando a la fuerza bruta pura frente a contraseñas de 12+ caracteres con charset amplio — de ahí el peso que el pentester debe dar a detectar políticas de contraseña débiles antes de lanzar un ataque por fuerza bruta pura.

### Tipos de ataque

| Método                  | Descripción                                                               | Cuándo usarlo                                                                         |
| ----------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Simple Brute Force      | Prueba todas las combinaciones de un charset y rango de longitud          | Sin información previa sobre la contraseña y con recursos computacionales suficientes |
| Dictionary Attack       | Wordlist precompilada de palabras/contraseñas comunes (ej. `rockyou.txt`) | El objetivo probablemente usa una contraseña débil o común                            |
| Hybrid Attack           | Diccionario + variaciones (añadir números/símbolos al final)              | El objetivo puede usar una versión modificada de una contraseña común                 |
| Credential Stuffing     | Credenciales filtradas de una brecha, probadas en otros servicios         | Hay credenciales filtradas disponibles y se sospecha reutilización de contraseñas     |
| Password Spraying       | Pocas contraseñas comunes contra muchos usuarios                          | Existen políticas de bloqueo de cuenta y se quiere evitar detección                   |
| Rainbow Table Attack    | Tablas precalculadas de hashes para revertirlos                           | Hay que crackear muchos hashes y se dispone de espacio de almacenamiento              |
| Reverse Brute Force     | Una contraseña fija contra múltiples usuarios                             | Fuerte sospecha de que una contraseña se reutiliza en varias cuentas                  |
| Distributed Brute Force | Carga repartida entre varias máquinas                                     | La clave es muy compleja y una sola máquina no llega en tiempo razonable              |

### Uso en pentesting

El brute forcing se emplea estratégicamente cuando:

* Otras vías han fallado (explotación de vulnerabilidades, ingeniería social).
* Las políticas de contraseñas del objetivo son débiles.
* Se buscan cuentas específicas, especialmente con privilegios elevados.

### Anatomía de una contraseña segura (NIST)

| Propiedad    | Detalle                                                                                                                                                                                        |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Longitud     | Mínimo 12 caracteres; cada carácter extra aumenta exponencialmente las combinaciones (26^6 ≈ 300M vs 26^8 ≈ 200 mil M)                                                                         |
| Complejidad  | Ampliar el charset (minúsculas+mayúsculas+números+símbolos) sube las posibilidades por posición (26 → 52 → más); NIST ya prioriza longitud/passphrases sobre esto, pero sigue siendo relevante |
| Unicidad     | Sin reutilización entre cuentas; así se compartimenta el daño si una se filtra                                                                                                                 |
| Aleatoriedad | Sin palabras de diccionario, datos personales ni frases comunes — evita caer en wordlists de atacantes                                                                                         |

### Debilidades comunes

| Debilidad                                   | Riesgo                                                |
| ------------------------------------------- | ----------------------------------------------------- |
| Contraseñas cortas (< 8 caracteres)         | Espacio de combinaciones pequeño                      |
| Palabras/frases comunes                     | Vulnerable a dictionary attack                        |
| Información personal (fechas, mascotas...)  | Fácil de adivinar si hay OSINT disponible             |
| Reutilización entre cuentas                 | Una filtración compromete todas las cuentas asociadas |
| Patrones predecibles (`qwerty`, `p@ssw0rd`) | Ya catalogados en wordlists de atacantes              |

### Políticas de contraseñas

Requisitos típicos: longitud mínima, complejidad de caracteres, caducidad periódica, historial (no reutilizar contraseñas anteriores). Políticas demasiado rígidas empujan a malas prácticas (contraseñas apuntadas, variaciones mínimas) — hay que equilibrar seguridad y usabilidad.

### Credenciales por defecto

Contraseñas y usuarios preestablecidos de fábrica en dispositivos/software, ampliamente documentados y fáciles de explotar. Reducen drásticamente el espacio de búsqueda; en muchos casos ni hace falta fuerza bruta real.

| Dispositivo/Fabricante | Usuario | Contraseña | Tipo                   |
| ---------------------- | ------- | ---------- | ---------------------- |
| Linksys Router         | admin   | admin      | Router inalámbrico     |
| D-Link Router          | admin   | admin      | Router inalámbrico     |
| Netgear Router         | admin   | password   | Router inalámbrico     |
| TP-Link Router         | admin   | admin      | Router inalámbrico     |
| Cisco Router           | cisco   | cisco      | Router de red          |
| Asus Router            | admin   | admin      | Router inalámbrico     |
| Belkin Router          | admin   | password   | Router inalámbrico     |
| Zyxel Router           | admin   | 1234       | Router inalámbrico     |
| Samsung SmartCam       | admin   | 4321       | Cámara IP              |
| Hikvision DVR          | admin   | 12345      | DVR                    |
| Axis IP Camera         | root    | pass       | Cámara IP              |
| Ubiquiti UniFi AP      | ubnt    | ubnt       | Access Point           |
| Canon Printer          | admin   | admin      | Impresora de red       |
| Honeywell Thermostat   | admin   | 1234       | Termostato inteligente |
| Panasonic DVR          | admin   | 12345      | DVR                    |

Nota: conocer el usuario ya es media batalla — si además es predecible (`admin`, `root`), el ataque se reduce solo a la contraseña. SecLists mantiene `top-usernames-shortlist.txt` para esto. Cambiar solo la contraseña y dejar el usuario por defecto sigue reduciendo la superficie de ataque a explotar.

### Impacto para el pentester

* La fortaleza real de las contraseñas del objetivo determina el éxito potencial del ataque.
* La complejidad esperada condiciona la herramienta/metodología: diccionario simple vs. híbrido más sofisticado.
* El tiempo y potencia de cómputo necesarios se planifican según esa complejidad.
* Las credenciales por defecto son el punto de entrada más rápido cuando existen.

### Referencias cruzadas

* `hydra` — ataques remotos de fuerza bruta contra servicios de red.
* `password-spraying-stuffing-y-credenciales-por-defecto` — desarrollo de spraying, stuffing y credenciales por defecto.
* `tecnicas-de-descifrado` — John the Ripper, Hashcat y ataques offline sobre hashes.
