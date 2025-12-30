# Path de linux

El path es una variable de entorno en la que esta la lista de directorios en la que el sistema busca los programas.

Todos los programas que esten en uno de los directorios del PATH se pueden ejecutar desde cualquier parte de sistema.



#### Añadir un directorio al PATH&#x20;

Para añadir un directorio al path tienes que exportar la ruta del directorio a la variable de entorno, pero esto solo es temporal, cuando reinicies el sistema se eliminara del PATH.

{% code title="Ejemplo" %}
```bash
export PATH="$PATH:/ruta/al/directorio"
```
{% endcode %}

Si quieres hacerlo permanentemente tienes que editar el .bashrc en el caso de que estes usando bash y añadir una linea igual a la anterior.

{% code title=".bashrc" %}
```
# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
# Añadir snap al PATH
export PATH=$PATH:/snap/bin

```
{% endcode %}
