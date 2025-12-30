# bypass de otros filtros incluidos en la lista negra

Además de los operadores de inyección y los caracteres espaciales, un carácter que suele figurar en la lista negra es la barra (`/`) o barra invertida (`\`), ya que es necesario especificar directorios en Linux o Windows. Podemos utilizar varias técnicas para producir cualquier carácter que queramos evitando el uso de caracteres incluidos en la lista negra.

***

### Linux

Hay muchas técnicas que podemos utilizar para tener cortes en nuestra carga útil. Una de esas técnicas la podemos utilizar para reemplazar barras (o cualquier otro carácter) usando las variables de entorno de Linux como hicimos con `${IFS}`. Mientras `${IFS}` se reemplaza directamente con un espacio, no existe tal variable de entorno para barras o punto y coma. Sin embargo, estos caracteres se pueden extraer de una variable de entorno y podemos especificar `start` y `length` de nuestra cadena para que coincida exactamente con este carácter.

Por ejemplo, si observamos la variable de entorno `$PATH` en Linux, puede verse así:

```shell-session
OsmanMartinez@htb[/htb]$ echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games
```

Entonces, si comenzamos en el `0` carácter, y solo tomamos una subcadena de longitud `1`, terminaremos con solo el `/` carácter que podemos utilizar en nuestra carga útil:

```shell-session
OsmanMartinez@htb[/htb]$ echo ${PATH:0:1}

/
```

{% hint style="info" %}
Nota: cuando use este método en la carga útil real no es necesario anteponer `echo` — aquí lo usamos solo para mostrar el carácter generado.
{% endhint %}

Podemos hacer lo mismo con las variables de entorno `$HOME` o `$PWD`. También podemos utilizar el mismo concepto para obtener un carácter de punto y coma, para utilizarlo como operador de inyección. Por ejemplo, el siguiente comando nos da un punto y coma:

```shell-session
OsmanMartinez@htb[/htb]$ echo ${LS_COLORS:10:1}

;
```

<details>

<summary>Ejercicio (Linux)</summary>

Trate de comprender cómo el comando anterior resultó en un punto y coma y luego úselo en la carga útil para usarlo como operador de inyección. Sugerencia: el comando `printenv` imprime todas las variables de entorno en Linux, por lo que puede ver cuáles pueden contener caracteres útiles y luego intentar reducir la cadena a ese carácter únicamente.

Por ejemplo, intente usar variables de entorno para agregar un punto y coma y un espacio a la carga útil: `127.0.0.1${LS_COLORS:10:1}${IFS}` y vea si puede omitir el filtro.

</details>

Como ejemplo visual, aquí hay una captura relacionada:\
![Captura de pantalla de una interfaz de aplicación web que muestra una solicitud POST a 127.0.0.1 con encabezados y una carga útil que contiene una posible inyección de comando. La sección de respuesta muestra código HTML para un formulario 'Host Checker', que permite la entrada de IP y muestra resultados de ping para 127.0.0.1.](../../.gitbook/assets/cmdinj_filters_spaces_5.jpg)

Como podemos ver, esta vez también evitamos con éxito el filtro de caracteres.

***

### Windows

El mismo concepto también funciona en Windows. Por ejemplo, para producir una barra en la línea de comandos de Windows (CMD), podemos extraer una subcadena de una variable de entorno (`%HOMEPATH%` -> `\Users\htb-student`), especificar una posición inicial (`~6` -> `\htb-student`), y finalmente especificar una posición final negativa, que en este caso es la longitud del nombre de usuario `htb-student` (`-11` -> `\`):

```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%

\
```

Podemos lograr lo mismo usando las mismas variables en Windows PowerShell. Con PowerShell, una cadena se trata como un arreglo de caracteres, por lo que debemos especificar el índice del carácter que necesitamos. Como solo necesitamos un carácter, no tenemos que especificar posiciones de inicio y fin:

```powershell-session
PS C:\htb> $env:HOMEPATH[0]

\


PS C:\htb> $env:PROGRAMFILES[10]
PS C:\htb>
```

También podemos utilizar el comando `Get-ChildItem Env:` en PowerShell para listar todas las variables de entorno y luego elegir una de ellas para producir el carácter que necesitamos. Try to be creative and find different commands to produce similar characters.

<details>

<summary>Ejercicio (Windows)</summary>

Intente usar variables de entorno diferentes en CMD o PowerShell para producir `/` o `\` sin escribir directamente el carácter. Explore variables como `%HOMEDRIVE%`, `%HOMEPATH%`, `%USERPROFILE%` y `$env:...` en PowerShell para identificar dónde está el carácter necesario y cómo extraerlo.

</details>

***

### Cambio de carácter (shifting characters)

Existen otras técnicas para producir los caracteres requeridos sin utilizarlos, como "shifting characters". Por ejemplo, el siguiente comando de Linux cambia el carácter por el que pasamos `1`. Entonces, todo lo que tenemos que hacer es encontrar el carácter en la tabla ASCII que está justo antes de nuestro carácter necesario (puede obtenerlo con `man ascii`), luego agregarlo en lugar de `[` en el siguiente ejemplo. De esta forma, el último carácter impreso sería el que necesitamos:

```shell-session
OsmanMartinez@htb[/htb]$ man ascii     # \ is on 92, before it is [ on 91
OsmanMartinez@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)

\
```

Podemos utilizar comandos de PowerShell para lograr el mismo resultado en Windows, aunque pueden ser bastante más largos que los de Linux.

<details>

<summary>Ejercicio (Cambio de carácter)</summary>

Intente utilizar la técnica de shifting characters para producir el carácter punto y coma `;`. Primero busque el carácter anterior en la tabla ASCII y luego úselo en el comando anterior para generar `;`.

</details>
