# Ataque Samba Relay

{% stepper %}
{% step %}
### Requisitos previos

Para hacer este ataque, tenemos que tener el DC y la máquina cliente conectadas y bien configuradas dentro del dominio. Una vez hecho esto, abrimos Kali Linux y nos ubicamos en la ubicación de la herramienta llamada responder.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Ejecutar responder

Lo ejecutamos con estos parámetros:

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Envenenamiento del tráfico

Responder se pone a envenenar el tráfico:

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Forzar una petición desde el cliente

Ahora, si desde la máquina cliente se intenta acceder a un recurso compartido que no existe, vamos a obtener el hash con las credenciales de dicha máquina.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Hash interceptado en Kali

Una vez hecho este intento, en la máquina Kali Linux desde responder habremos interceptado el hash con las credenciales de este usuario:

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Interceptar hash del administrador desde el DC

También podemos interceptar de la misma forma el hash del usuario administrador si esta petición se hace desde el DC:

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Ataque de fuerza bruta con John the Ripper

Con John the Ripper podemos intentar un ataque de fuerza bruta a cualquiera de estos hashes; si la contraseña es débil, funcionará.


{% endstep %}
{% endstepper %}
