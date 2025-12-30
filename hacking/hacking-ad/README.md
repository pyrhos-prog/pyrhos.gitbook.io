# Introducción

## **Hacking Active Directory**

El hacking de Directorio Activo es el conjunto de técnicas utilizadas por un atacante para comprometer este servicio fundamental de Microsoft.

El Directorio Activo es el "cerebro" de la mayoría de las redes corporativas que usan Windows. Gestiona y centraliza la identidad y la autenticación de todos los usuarios, equipos, permisos y políticas de seguridad de la organización.

### &#x20;**Cual es el objetivo?**

El objetivo final de un atacante al hackear el Directorio Activo es casi siempre el mismo: obtener el control total de la red. Esto se conoce como "conseguir el Administrador de Dominio"

#### **Una vez que un atacante tiene estas credenciales, puede:**

* Acceder a cualquier ordenador o servidor de la red.
* Leer, modificar o borrar cualquier dato.
* Robar contraseñas de todos los usuarios.
* Desplegar un ransomware en toda la empresa.
* Crear cuentas de administrador fantasma para mantener el acceso.

### **Procedimiento del Hacking AD**

El hacking de AD no suele ser un ataque directo desde el exterior, sino que ocurre _después_ de que un atacante ha logrado acceder a un solo ordenador dentro de la red.

#### **El atacante realiza los siguientes pasos:**

{% stepper %}
{% step %}
### Reconocimiento

mapear el Directorio Activo para entender la estructura: usuarios, grupos, administradores y en donde estan conectados.
{% endstep %}

{% step %}
### Movimiento Lateral y Escalada de Privilegios

&#x20;El atacante intenta  pivotar de un ordenador a otro, robando credenciales en el camino, hasta encontrar las de un usuario con más privilegios.
{% endstep %}
{% endstepper %}

#### **Algunas de las técnicas más comunes incluyen:**

{% tabs %}
{% tab title="Robo de Credenciales" %}
Usar herramientas como Mimikatz para extraer contraseñas guardadas en la memoria de un ordenador.
{% endtab %}

{% tab title="Kerberoasting" %}
Un ataque que se aprovecha de cuentas de servicio (usadas por aplicaciones) que a menudo tienen contraseñas débiles. El atacante solicita un "ticket" de autenticación y luego intenta descifrar la contraseña de esa cuenta fuera de la red.
{% endtab %}

{% tab title="Explotación de Malas Configuraciones" %}
Buscar fallos comunes, como permisos excesivos en cuentas de usuario normales, contraseñas débiles que nunca caducan o sistemas que no han sido actualizados (parcheados).
{% endtab %}
{% endtabs %}
