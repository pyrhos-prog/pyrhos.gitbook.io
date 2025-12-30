# Detección de inyección de comandos

El proceso de detección de vulnerabilidades básicas de inyección de comandos del sistema operativo es el mismo proceso para explotar dichas vulnerabilidades. Intentamos agregar nuestro comando a través de varios métodos de inyección. Si la salida del comando cambia con respecto al resultado habitual previsto, hemos explotado con éxito la vulnerabilidad. Esto puede no ser cierto para vulnerabilidades de inyección de comandos más avanzadas porque podemos utilizar varios métodos de fuzzing o revisiones de código para identificar posibles vulnerabilidades de inyección de comandos. Luego podremos construir gradualmente nuestra carga útil hasta lograr la inyección de comandos. Este módulo se centrará en inyecciones de comandos básicas, donde controlamos la entrada del usuario que se utiliza directamente en la ejecución de una función de comando del sistema sin ninguna desinfección.

Para demostrarlo, utilizaremos el ejercicio que se encuentra al final de esta sección.

## Detección de inyección de comandos

Cuando visitamos la aplicación web en el siguiente ejercicio, vemos un Host Checker utilidad que parece pedirnos una IP para comprobar si está activa o no:

![Imagen de una interfaz de Host Checker con un campo de texto denominado 'Ingrese una dirección IP' que contiene '127.0.0.1' y un botón 'Verificar' a continuación.](<../../.gitbook/assets/cmdinj_basic_exercise_1 (1).jpg>)

Podemos intentar ingresar la IP del host local `127.0.0.1` para comprobar la funcionalidad y, como se esperaba, devuelve la salida del `ping` comando que nos dice que el host local está realmente vivo:

![Interfaz Host Checker con un campo de texto para ingresar una dirección IP, se ingresó '127.0.0.1', un botón 'Verificar' y los resultados de ping se muestran a continuación.](<../../.gitbook/assets/cmdinj_basic_exercise_2 (1).jpg>)

Aunque no tenemos acceso al código fuente de la aplicación web, podemos adivinar con confianza que la IP que ingresamos va a un `ping` comando ya que la salida que recibimos sugiere eso. Como el resultado muestra un solo paquete transmitido en el comando ping, el comando utilizado puede ser el siguiente:

{% code title="Comando probable" %}
```bash
ping -c 1 OUR_INPUT
```
{% endcode %}

Si nuestra entrada no se desinfecta y se escapa antes de usarse con el `ping` comando, es posible que podamos inyectar otro comando arbitrario. Entonces, intentemos ver si la aplicación web es vulnerable a la inyección de comandos del sistema operativo.

## Métodos de inyección de comandos

Para inyectar un comando adicional al previsto, podemos utilizar cualquiera de los siguientes operadores:

| Operador de inyección | Carácter de inyección | Carácter codificado en URL | Comando ejecutado                                         |
| --------------------- | --------------------: | -------------------------- | --------------------------------------------------------- |
| Semicolon             |                   `;` | `%3b`                      | Ambos                                                     |
| Nueva línea           |                  `\n` | `%0a`                      | Ambos                                                     |
| Antecedentes          |                   `&` | `%26`                      | Ambos (la segunda salida generalmente se muestra primero) |
| Tubería               |                  `\|` | `%7c`                      | Ambos (solo se muestra la segunda salida)                 |
| Y                     |                  `&&` | `%26%26`                   | Ambos (sólo si el primero tiene éxito)                    |
| O                     |                  `\|` | `%7c%7c`                   | Segundo (sólo si el primero falla)                        |
| Sub-Shell             |             `` ` ` `` | `%60%60`                   | Ambos **(Solo Linux)**                                    |
| Sub-Shell             |                 `$()` | `%24%28%29`                | Ambos **(Solo Linux)**                                    |

Podemos usar cualquiera de estos operadores para inyectar otro comando. Escribimos nuestra entrada esperada (por ejemplo, una IP), luego usamos cualquiera de los operadores anteriores y después escribimos nuestro nuevo comando.

En general, para la inyección de comandos básicos, todos estos operadores se pueden utilizar para inyecciones de comandos independientemente del lenguaje de la aplicación web, framework o servidor back-end. Entonces, si estamos inyectando en una aplicación `PHP` ejecutándose en un servidor `Linux`, o una aplicación `.Net` ejecutándose en un servidor `Windows` back-end, o una aplicación `NodeJS` ejecutándose en un servidor `macOS` back-end, nuestras inyecciones deberían funcionar de todos modos.

{% hint style="warning" %}
Nota: La única excepción puede ser el punto y coma `;`, que no funcionará si el comando se estaba ejecutando con Windows Command Line (CMD), pero aún funcionaría si se estuviera ejecutando con Windows PowerShell.
{% endhint %}

En la siguiente sección, intentaremos utilizar uno de los operadores de inyección anteriores para explotar el `Host Checker` ejercicio.
