# Omitir Otros caracteres

Además de los operadores de inyección y los caracteres de espacio, un carácter muy común en la lista negra es el carácter de barra diagonal (/) o barra invertida (), ya que es necesario especificar directorios en Linux o Windows. Podemos utilizar varias técnicas para producir cualquier carácter que queramos evitando el uso de caracteres en la lista negra.

***

## Linux

Hay muchas técnicas que podemos utilizar para tener barras diagonales en nuestra carga útil. Una de esas técnicas que podemos usar para reemplazar barras diagonales (/) es a través de variables de entorno, como lo hicimos con ${IFS}. Si bien ${IFS} se reemplaza directamente con un espacio, no existe tal variable de entorno para barras diagonales o punto y coma literal. Sin embargo, estos caracteres se pueden encontrar dentro de otras variables de entorno, y podemos extraer subcadenas de nuestra cadena para obtener exactamente ese carácter: empezando en una posición y tomando una longitud determinada.

Por ejemplo, si nos fijamos en la variable de entorno `$PATH`, puede tener un aspecto similar al siguiente:

```shell-session
zunderrubb@htb[/htb]$ echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games
```

Si empezamos en el carácter 0 y solo tomamos una longitud 1, terminaremos con solo el carácter `/`, que podemos usar en nuestra carga útil:

```shell-session
zunderrubb@htb[/htb]$ echo ${PATH:0:1}

/
```

{% hint style="info" %}
Nota: Cuando usamos el comando anterior en nuestra carga útil, no agregaremos `echo`, ya que solo lo estamos mostrando aquí para ilustrar el carácter de salida.
{% endhint %}

También podemos hacer lo mismo con otras variables de entorno como `$HOME` o `$PWD`. El mismo concepto puede utilizarse para obtener un carácter de punto y coma (`;`), que se usará como operador de inyección. Por ejemplo, el siguiente comando obtiene un punto y coma:

```shell-session
zunderrubb@htb[/htb]$ echo ${LS_COLORS:10:1}

;
```

Ejercicio: Intente comprender cómo el comando anterior dio como resultado un punto y coma y, a continuación, utilícelo en la carga útil para usarlo como operador de inyección. Sugerencia: El comando `printenv` imprime todas las variables de entorno en Linux, por lo que puede ver cuáles pueden contener caracteres útiles y luego intentar reducir la cadena solo a ese carácter.

Por lo tanto, intentemos usar variables de entorno para agregar un punto y coma y un espacio a nuestra carga útil (por ejemplo, `127.0.0.1${LS_COLORS:10:1}${IFS}`) y veamos si podemos omitir el filtro:

![Operador de filtro](../../.gitbook/assets/cmdinj_filters_spaces_5.jpg)

Como podemos ver, esta vez también hemos evitado con éxito el filtro de caracteres.

***

## Windows

El mismo concepto funciona también en Windows. Por ejemplo, para producir una barra invertida en CMD, podemos usar una variable de Windows (`%HOMEPATH%` -> `\Users\htb-student`), y luego especificar una posición inicial (`~6`) y una posición final negativa (por ejemplo `-11`, que representa la longitud a recortar). Ejemplo:

```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%

\
```

Podemos lograr lo mismo usando las mismas variables en PowerShell. En PowerShell, una cadena se trata como una matriz de caracteres, por lo que tenemos que especificar el índice del carácter que necesitamos. Como solo necesitamos un carácter, no tenemos que especificar las posiciones inicial y final:

```powershell-session
PS C:\htb> $env:HOMEPATH[0]

\


PS C:\htb> $env:PROGRAMFILES[10]
PS C:\htb>
```

También podemos usar el comando `Get-ChildItem Env:` para imprimir todas las variables de entorno y luego elegir una de ellas para producir el carácter que necesitamos.

{% hint style="info" %}
Try to be creative and find different commands to produce similar characters.
{% endhint %}

***

## Cambio de carácter

Existen otras técnicas para producir los caracteres requeridos sin usarlos literalmente, como desplazar (shifting) caracteres. Por ejemplo, el siguiente enfoque en Linux desplaza el carácter por el que pasamos. Todo lo que tenemos que hacer es encontrar el carácter en la tabla ASCII que está justo antes de nuestro carácter necesario (puede consultarse con `man ascii`), y luego transformarlo para obtener el carácter objetivo. En este ejemplo, `[` (91) está justo antes de `\` (92):

```shell-session
zunderrubb@htb[/htb]$ man ascii     # \ is on 92, before it is [ on 91
zunderrubb@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)

\
```

Podemos usar comandos de PowerShell para lograr el mismo resultado en Windows, aunque pueden ser bastante más largos que los de Linux.

***

<details>

<summary>Ejercicio: usar variables de entorno para omitir filtros</summary>

Intente:

* Ejecutar `printenv` o `Get-ChildItem Env:` para listar variables de entorno.
* Buscar variables que contengan `/`, `\`, `;` u otros caracteres útiles.
* Aplicar subcadena (por ejemplo `${VAR:pos:len}` en Bash o `$env:VAR[index]` en PowerShell) para extraer solo el carácter deseado.
* Construir una carga útil que combine el carácter obtenido con operadores (`;`, espacios via `${IFS}`, etc.) para evadir filtros.

</details>
