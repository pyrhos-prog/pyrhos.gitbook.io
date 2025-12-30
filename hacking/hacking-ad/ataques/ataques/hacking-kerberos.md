# Hacking Kerberos

Kerberos es un protocolo de autenticación que se utiliza en sistemas Windows para proporcionar seguridad en la autenticación de usuarios y servicios. Cuando un usuario se autentica en un sistema Windows, se solicita un ticket de servicio para el servicio al que está intentando acceder. Este ticket de servicio contiene una clave secreta que se utiliza para cifrar las comunicaciones entre el usuario y el servicio.

!\[\[Pasted image 20230218134749.png]]

***

{% stepper %}
{% step %}
### Resumen del ataque (Kerberoasting)

Este ataque consiste en capturar un Ticket Granting Service (TGS), el cual es un hash con las credenciales del administrador en caso de que el usuario SVC\_TGS sea kerberoasteable. Para hacer este ataque debemos tener credenciales válidas del usuario kerberoasteable (las tenemos del SVC\_TGS).

{% hint style="info" %}
Antes de solicitar TGS, es importante sincronizar el reloj con la misma hora que el domain controller. Para ello se puede usar la herramienta ntpdate.
{% endhint %}

!\[\[Pasted image 20230126103119.png]]
{% endstep %}

{% step %}
### Comprobar si el usuario es kerberoasteable

Para comprobar si podemos lanzar este ataque podemos solicitar un TGS con este comando y veremos que el usuario Administrator es kerberoasteable:

!\[\[Pasted image 20230126104404.png]]
{% endstep %}

{% step %}
### Solicitar el TGS con la opción -request

Viendo que podemos solicitar un TGS con las credenciales de este usuario, podemos ejecutar el mismo comando pero pasando la opción -request:

!\[\[Pasted image 20230126104553.png]]
{% endstep %}

{% step %}
### Extraer y guardar el hash

Una vez obtenido este hash, podemos tratar de crackearlo para que nos proporcione la contraseña del usuario administrator. Copia todo este hash y guárdalo en un archivo .txt:

!\[\[Pasted image 20230126104909.png]]
{% endstep %}

{% step %}
### Cracking del hash

Lanzamos el ataque con John para crackear este hash y obtener la contraseña:

!\[\[Pasted image 20230126105336.png]]
{% endstep %}

{% step %}
### Validación de credenciales y acceso

Validamos con crackmapexec que esta contraseña sirve para el usuario Administrator; y vemos que sí:

!\[\[Pasted image 20230126105512.png]]

A continuación, nos conectamos de manera remota con estas credenciales utilizando psexec.py y conseguimos una CMD como el usuario Administrator:

!\[\[Pasted image 20230126105848.png]]
{% endstep %}
{% endstepper %}

***

\[\[MAQUINA ACTIVE (Recursos Compartidos de SMB y listar recursos compartidos con smbmap y desencriptar contraseñas con gpp-decrypt - fichero groups.xml, smbclient, smbmap y kerberoasting con TGS)]]

**KERBRUTE**

Kerbrute es una herramienta para realizar un ataque de fuerza bruta sobre Kerberos en una máquina víctima. Este es su repositorio:

!\[\[Pasted image 20230425150757.png]]

Procedemos con su instalación: en primer lugar clonamos el repositorio y ejecutamos el fichero requirements para instalar las dependencias:

!\[\[Pasted image 20230425150924.png]]

Con el comando kerbrute ya vemos las instrucciones de cómo utilizarlo:

!\[\[Pasted image 20230425151011.png]]

Ahora necesitaremos un diccionario de usuarios y otro de contraseñas (pueden usarse los proporcionados en la web de TryHackMe):

!\[\[Pasted image 20230425154502.png]] !\[\[Pasted image 20230425154603.png]]

**BÚSQUEDA DE USUARIOS DOMINIO**

Este es el parámetro que debemos usar para hacer una búsqueda de usuarios válidos del dominio:

{% code title="Comando kerbrute" %}
```bash
python3 kerbrute.py -users userlist.txt -passwords passwordlist.txt -domain spookysec.local -t 100
```
{% endcode %}

En el resultado vemos varios usuarios, aunque hay uno de ellos que no requiere autenticación:

!\[\[Pasted image 20230425154919.png]]
