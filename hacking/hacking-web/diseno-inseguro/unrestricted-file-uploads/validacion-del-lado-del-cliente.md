# Validación del lado del cliente

Muchas aplicaciones web solo dependen del código JavaScript del front-end para validar el formato de archivo seleccionado antes de cargarlo y no lo cargarían si el archivo no está en el formato requerido (por ejemplo, no es una imagen).

Sin embargo, como la validación del formato de archivo se realiza en el lado del cliente, podemos evitarla fácilmente interactuando directamente con el servidor, omitiendo por completo las validaciones del front-end. También podemos modificar el código front-end a través de las herramientas de desarrollo de nuestro navegador para deshabilitar cualquier validación existente.

### Validación del lado del cliente

El ejercicio al final de esta sección muestra una descripción básica `Profile Image` funcionalidad, que se ve con frecuencia en aplicaciones web que utilizan funciones de perfil de usuario, como aplicaciones web de redes sociales:

![Actualice la imagen de perfil con el botón de carga.](../../.gitbook/assets/file_uploads_profile_image_upload.jpg)

Sin embargo, esta vez, cuando obtenemos el cuadro de diálogo de selección de archivos, no podemos ver nuestro `PHP` scripts (o puede estar en gris), ya que el cuadro de diálogo parece estar limitado únicamente a formatos de imagen:

![Cuadro de diálogo de carga de archivos con opciones para tipos de archivos: jpg, jpeg, png.](../../.gitbook/assets/file_uploads_select_file_types.jpg)

Todavía podemos seleccionar el `All Files` opción para seleccionar nuestra `PHP` script de todos modos, pero cuando lo hacemos, recibimos un mensaje de error que dice (`Only images are allowed!`), și el `Upload` El botón se desactiva:

![Actualice la imagen de perfil con el botón de carga; solo se permiten imágenes.](../../.gitbook/assets/file_uploads_select_denied.jpg)

Esto indica algún tipo de validación del tipo de archivo, por lo que no podemos simplemente cargar un shell web a través del formulario de carga como lo hicimos en la sección anterior. Afortunadamente, toda la validación parece realizarse en el front-end, ya que la página nunca se actualiza ni envía ninguna solicitud HTTP después de seleccionar nuestro archivo. Por lo tanto, deberíamos poder tener control total sobre estas validaciones del lado del cliente.

Cualquier código que se ejecute en el lado del cliente está bajo nuestro control. Si bien el servidor web es responsable de enviar el código front-end, la representación y ejecución del código front-end se realiza dentro de nuestro navegador. Si la aplicación web no aplica ninguna de estas validaciones en el back-end, deberíamos poder cargar cualquier tipo de archivo.

Como se mencionó anteriormente, para eludir estas protecciones, podemos: `modify the upload request to the back-end server`, o podemos `manipulate the front-end code to disable these type validations`.

{% stepper %}
{% step %}
### Modificación de solicitud de back-end

Comencemos examinando una solicitud normal con `Burp`. Cuando seleccionamos una imagen, vemos que se refleja como nuestra imagen de perfil y cuando hacemos clic en `Upload`, nuestra imagen de perfil se actualiza y persiste a través de actualizaciones. Esto indica que nuestra imagen fue cargada en el servidor, que ahora nos la muestra nuevamente:

![Actualice la imagen de perfil con el botón de carga.](../../.gitbook/assets/file_uploads_normal_request.jpg)

Si capturamos la solicitud de carga con `Burp`, vemos la siguiente solicitud enviada por la aplicación web:

![Solicitud HTTP POST a/upload.php con archivo HTB.png, tipo de contenido image/png.](../../.gitbook/assets/file_uploads_image_upload_request.jpg)

La aplicación web parece estar enviando una solicitud de carga HTTP estándar a `/upload.php`. De esta manera, ahora podemos modificar esta solicitud para satisfacer nuestras necesidades sin tener restricciones de validación de tipo front-end. Si el servidor back-end no valida el tipo de archivo cargado, entonces teóricamente deberíamos poder enviar cualquier tipo/contenido de archivo y se cargaría en el servidor.

Las dos partes importantes de la solicitud son `filename="HTB.png"` y el contenido del archivo al final de la solicitud. Si modificamos el `filename` a `shell.php` y modificar el contenido al shell web que usamos en la sección anterior; estaríamos cargando un `PHP` shell web en lugar de una imagen.

Entonces, capturemos otra solicitud de carga de imagen y luego modifiquémosla en consecuencia:

![Solicitud HTTP POST a /upload.php con archivo shell.php, respuesta 200 OK, archivo cargado exitosamente.](../../.gitbook/assets/file_uploads_modified_upload_request.jpg)

Nota: También podemos modificar el `Content-Type` del archivo cargado, aunque esto no debería jugar un papel importante en esta etapa, por lo que lo mantendremos sin modificaciones.

Como podemos ver, nuestra solicitud de carga se realizó y obtuvimos `File successfully uploaded` en la respuesta. Entonces, ahora podemos visitar nuestro archivo cargado e interactuar con él y obtener la ejecución remota de código.
{% endstep %}

{% step %}
### Deshabilitar la validación del frontend

Otro método para evitar las validaciones del lado del cliente es mediante la manipulación del código del front-end. Como estas funciones se procesan completamente dentro de nuestro navegador web, tenemos control total sobre ellas. Entonces, podemos modificar estos scripts o deshabilitarlos por completo. Luego, podemos usar la funcionalidad de carga para cargar cualquier tipo de archivo sin necesidad de utilizar `Burp` para capturar y modificar nuestras solicitudes.

Para comenzar, podemos hacer clic en \[CTRL+SHIFT+C] para alternar el navegador `Page Inspector`, y luego haga clic en la imagen de perfil, que es donde activamos el selector de archivos para el formulario de carga:

![Actualice la interfaz de la imagen de perfil con el botón de carga y la vista de código HTML.](../../.gitbook/assets/file_uploads_element_inspector.jpg)

Esto resaltará la siguiente entrada de archivo HTML en línea `18`:

Cod: html

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```

Aquí vemos que la entrada del archivo especifica (`.jpg,.jpeg,.png`) como los tipos de archivos permitidos dentro del cuadro de diálogo de selección de archivos. Sin embargo, podemos modificar esto fácilmente y seleccionar `All Files` como hicimos antes, por lo que no es necesario cambiar esta parte de la página.

La parte más interesante es `onchange="checkFile(this)"`, que parece ejecutar un código JavaScript cada vez que seleccionamos un archivo, que parece estar realizando la validación del tipo de archivo. Para obtener los detalles de esta función, podemos ir al navegador `Console` haciendo clic en \[CTRL+SHIFT+K], y luego podemos escribir el nombre de la función (`checkFile`) para obtener sus detalles:

Cod: javascript

```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```

Lo clave que tomamos de esta función es donde verifica si la extensión del archivo es una imagen y, si no lo es, imprime el mensaje de error que vimos anteriormente (`Only images are allowed!`) y desactiva el `Upload` botón. Podemos agregar `PHP` como una de las extensiones permitidas o modificar la función para eliminar la verificación de extensión.

Afortunadamente, no necesitamos dedicarnos a escribir y modificar código JavaScript. Podemos eliminar esta función del código HTML ya que su uso principal parece ser la validación del tipo de archivo y eliminarla no debería romper nada.

Para ello, podemos volver a nuestro inspector, hacer clic nuevamente en la imagen de perfil, hacer doble clic en el nombre de la función (`checkFile`) en línea `18`, y eliminarlo:

![Formulario HTML para cargar imágenes de perfil, acepta jpg, jpeg, png.](../../.gitbook/assets/file_uploads_removed_js_function.jpg)

{% hint style="info" %}
También puedes hacer lo mismo para eliminar `accept=".jpg,.jpeg,.png"`, lo que debería hacer que seleccionar el `PHP` shell sea más fácil en el cuadro de diálogo de selección de archivos, aunque esto no es obligatorio.
{% endhint %}

Con el `checkFile` función eliminada de la entrada del archivo, deberíamos poder seleccionar nuestra `PHP` web shell a través del cuadro de diálogo de selección de archivos y cargarlo normalmente sin validaciones, similar a lo que hicimos en la sección anterior.

Nota: La modificación que hicimos al código fuente es temporal y no persistirá durante las actualizaciones de la página, ya que solo la estamos cambiando en el lado del cliente. Sin embargo, nuestra única necesidad es eludir la validación del lado del cliente, por lo que debería ser suficiente para este propósito.

Una vez que cargamos nuestro shell web usando cualquiera de los métodos anteriores y luego actualizamos la página, podemos usar el `Page Inspector` una vez más con \[CTRL+SHIFT+C], haga clic en la imagen de perfil y deberíamos ver la URL de nuestro shell web cargado:

Cod: html

```html
<img src="/profile_images/shell.php" class="profile-image" id="profile-image">
```

Si podemos hacer clic en el enlace anterior, llegaremos a nuestro shell web cargado, con el que podemos interactuar para ejecutar comandos en el servidor back-end:

![UID, GID y grupos configurados en 33 (www-data).](../../.gitbook/assets/file_uploads_php_manual_shell.jpg)

Nota: Los pasos que se muestran se aplican a Firefox, ya que otros navegadores pueden tener métodos ligeramente diferentes para aplicar cambios locales a la fuente, como el uso de `overrides` en Chrome.
{% endstep %}
{% endstepper %}
