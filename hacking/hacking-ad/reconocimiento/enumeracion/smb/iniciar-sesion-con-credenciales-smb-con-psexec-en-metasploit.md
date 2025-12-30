# Iniciar Sesión con Credenciales SMB con psexec en Metasploit

{% stepper %}
{% step %}
### Ataque de fuerza bruta contra SMB

Utilizamos un ataque de fuerza bruta contra el protocolo SMB y obtenemos las credenciales:

```bash
use auxiliary/scanner/smb/smb_login
```

!\[\[Pasted image 20230730161556.png]]
{% endstep %}

{% step %}
### Inicio de sesión con PsExec

Podemos usar el módulo de psexec para iniciar sesión con dichas credenciales:

```bash
use exploit/windows/smb/psexec
```

!\[\[Pasted image 20230730161744.png]]
{% endstep %}
{% endstepper %}
