# Permisos especiales - SUID ySGID

### SUID&#x20;

Cuando un archivo tiene el permiso SUID podemos ejecutarlo como si fueramos el usuario propietario, esto es peligroso porque podemos escalar privilegios si algunos archivos estan mal configurados.

Podemos activarlo de 2 maneras:

```
chmod u+s <archivo>
O
chmod 4*** <archivo>
```

### SGID

El permiso SGID se pone en los permisos del grupo, cuando esta en un archivo permite que cualquier usuario ejecute el archivo como si fuese miembro del grupo al que pertenece el archivo.

Podemos activarlo de 2 maneras:

```
chmod g+s <archivo>
O
chmod 2*** <archivo>
```

{% hint style="info" %}
Estos permisos se representan con una _**s minúscula**_ cuando está activo y posee permisos de ejecución, y con una _**S mayúscula**_ cuando está activo y no posee permisos de ejecución.
{% endhint %}

