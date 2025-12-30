# ldap

* Para enumerar LDAP tenemos varias opciones y lo mejor que podemos hacer si no sabemos cómo hacerlo sería chequear en HackTricks, ya que viene todo perfectamente contemplado.
* Podríamos probar con ldapsearch para enumerarlo todo. Podemos probar sin credenciales, pero si tenemos credenciales podríamos probar con estos comandos:

{% code title="Con credenciales" %}
```bash
ldapsearch -x -H ldap://10.10.11.174 -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb"
```
{% endcode %}

!\[\[Pasted image 20240723122701.png]]

* Esta forma la podemos usar si realiza sin contraseña ni usuario:

{% code title="Sin credenciales" %}
```bash
ldapsearch -x -H ldap://10.10.10.175 -b "DC=EGOTISTICAL-BANK,DC=LOCAL"
```
{% endcode %}
