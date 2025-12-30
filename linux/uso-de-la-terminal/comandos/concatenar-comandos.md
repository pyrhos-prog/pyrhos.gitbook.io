# Concatenar comandos

{% stepper %}
{% step %}
### Concatenación con "|"

El operador pipe "|" sirve para que la salida del primer comando se convierta en la entrada del segundo. De esta forma dos comandos se encadenan: primero se ejecuta el primero y su salida es procesada por el segundo.
{% endstep %}

{% step %}
```bash
pyrhos@debian:~/Documentos$ cat ping.log | grep "ping"
--- 192.168.1.1 ping statistics ---
```


{% endstep %}
{% endstepper %}
