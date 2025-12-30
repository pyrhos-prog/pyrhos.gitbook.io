# AS-REPRoasting

Es una técnica que consiste en encontrar usuarios que no requieren pre-autenticación de Kerberos. Esto significa que cualquiera puede enviar una petición AS\_REQ en nombre de uno de esos usuarios y recibir un mensaje AS\_REP válido. La respuesta contiene un fragmento cifrado con la clave del usuario (derivada de su contraseña). Para extraer y aprovechar estos AS\_REP se puede usar la herramienta GetNPUsers.py.

Por ejemplo, podríamos encontrar este tipo de usuarios utilizando \[\[Kerbrute]] !\[\[Pasted image 20230425154919.png]]

{% stepper %}
{% step %}
### Opción: Proporcionando un diccionario de usuarios

Proporciona un diccionario de usuarios (por ejemplo, un fichero llamado `users`) para que la herramienta pruebe múltiples cuentas.

!\[\[Pasted image 20230126114019.png]]
{% endstep %}

{% step %}
### Opción: Proporcionando un único usuario

Proporciona un único usuario que hayas podido identificar previamente para solicitar su AS\_REP.

!\[\[Pasted image 20230425155219.png]]
{% endstep %}
{% endstepper %}

\[\[MAQUINA ATTACKTIVE DIRECTORY (enum4linux, kerbrute enumerar usuarios, ASREPRoasting obtener TGT, john the ripper, smbclient para listar recursos compartidos, secredump dumpear hashes y pass the hash)]]
