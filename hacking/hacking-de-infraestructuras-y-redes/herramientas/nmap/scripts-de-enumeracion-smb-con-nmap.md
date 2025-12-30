# Scripts de enumeración SMB con nmap

### SCRIPT ENUMERAR SMB

{% stepper %}
{% step %}
Este script busca vulnerabilidades en smb:

{% code title="Comando" %}
```bash
nmap -p445 --script smb-protocols 192.168.0.48
```
{% endcode %}
{% endstep %}

{% step %}
También podemos saber la seguridad si permite iniciar sesión como invitado o no con este otro script, de tal forma que podríamos entrar sin proporcionar contraseña:

{% code title="Comando" %}
```bash
nmap -p445 --script smb-security-mode 192.168.0.48
```
{% endcode %}
{% endstep %}

{% step %}
Este script sirve listar recursos compartidos con smb con nmap usando el siguiente script:

{% code title="Comando" %}
```bash
nmap -p445 --script smb-enum-shares 192.168.0.48
```
{% endcode %}
{% endstep %}
{% endstepper %}



