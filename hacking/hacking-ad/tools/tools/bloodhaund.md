# BloodHaund

* Lo primero será:
* SharpHound.ps1: Se usa para recolectar toda la información del equipo y así poder llevártela al BloodHound.

!\[\[Pasted image 20240424204431.png]]

!\[\[Pasted image 20240723142638.png]]

* Comando para recopilar la información de un sistema de Active Directory:

```bash
bloodhound-python -c All -u ' ' -p ' ' -d support.htb -ns <ip> --zip
```

{% hint style="warning" %}
Comando para borrar todos los datos en Neo4j (usa con precaución):

```
MATCH (n) DETACH DELETE n;
```
{% endhint %}
