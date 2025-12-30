# Injeccion basica

\#commandInjection #comandos #injection

Tenemos un panel de login que cuando metemos la ip nos devuelve el resultado del comando ping por lo que vamos a intentar escapar el comando y meterle el código que nosotros necesitamos para conseguir ver información.

### Métodos de inyección de comandos

Para inyectar un comando adicional al deseado, podemos usar cualquiera de los siguientes operadores:

| **Operador de inyección** | **Carácter de inyección** | **Carácter codificado en URL** | **Comando ejecutado**                                     |
| ------------------------- | ------------------------- | ------------------------------ | --------------------------------------------------------- |
| Punto y coma              | `;`                       | `%3b`                          | Ambos                                                     |
| Nueva línea               | `\n`                      | `%0a`                          | Ambos                                                     |
| Fondo                     | `&`                       | `%26`                          | Ambos (la segunda salida generalmente se muestra primero) |
| Pipa                      | `\|`                      | `%7c`                          | Ambos (solo se muestra la segunda salida)                 |
| Y                         | `&&`                      | `%26%26`                       | Ambos (solo si primero tiene éxito)                       |
| O                         | `\|`                      | `%7c%7c`                       | Segundo (solo si falla el primero)                        |
| Sub-Carcasa               | \`\` \`                   | `%60%60`                       | Ambos (solo Linux)                                        |
| Sub-Carcasa               | `$()`                     | `%24%28%29`                    | Ambos (solo Linux)                                        |

{% hint style="warning" %}
Tenemos un problema ya que no podemos hacer una inyección de comandos desde la propia web porque el parámetro filtra el input y solo permite una IP. Para poder ejecutar el comando tiramos de Burp para interceptar la petición y colarle ese comando y así conseguir ejecución remota de comandos.
{% endhint %}

!\[\[Pasted image 20240328151023.png]]

Aquí vemos el filtro que impide que ejecutemos comandos desde el input del usuario

!\[\[Pasted image 20240328162020.png]]

Estos operadores se pueden utilizar para varios tipos de inyección, como inyecciones SQL, inyecciones LDAP, XSS, SSRF, XML, etc. Hemos creado una lista de los operadores más comunes que se pueden utilizar para las inyecciones:

| **Tipo de inyección**                                 | **Operadores**                                           |
| ----------------------------------------------------- | -------------------------------------------------------- |
| Inyección SQL                                         | `'`, `,`, `;`, `--`, `/* */`                             |
| Inyección de comandos                                 | `;`, `&&`                                                |
| Inyección LDAP                                        | `*`, `(`, `)`, `&`, `\|`                                 |
| Inyección XPath                                       | `'`, `or`, `and`, `not`, `substring`, `concat`, `count`  |
| Inyección de comandos del sistema operativo           | `;`, `&`, `\|`                                           |
| Inyección de código                                   | `'`, `;`, `--`, `/* */`, `$()`, `${}`, `#{}`, `%{}`, `^` |
| Recorrido de directorios/recorrido de ruta de archivo | `../`, `..\\`, `%00`                                     |
| Inyección de objetos                                  | `;`, `&`, `\|`                                           |
| Inyección XQuery                                      | `'`, `;`, `--`, `/* */`                                  |
| Inyección de código de shell                          | `\x`, `\u`, `%u`, `%n`                                   |
| Inyección de encabezado                               | `\n`, `\r\n`, `\t`, `%0d`, `%0a`, `%09`                  |
