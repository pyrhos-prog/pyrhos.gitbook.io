# Comando lsof + ejemplo de uso

{% stepper %}
{% step %}
### Ejecutar un servidor web con Python en el puerto 8080

Ejecutamos un servidor web con Python en el puerto 8080:

```
pyrhos@debian:~$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```
{% endstep %}

{% step %}
### Ver qué servicio está usando el puerto 8080

Con el comando lsof puedes ver qué servicio está ocupando el puerto 80 de mi equipo:

```bash
pyrhos@debian:~$ lsof -i:8080
COMMAND   PID   USER FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 17490 pyrhos 3u  IPv4  78843      0t0  TCP *:http-alt (LISTEN)

```


{% endstep %}

{% step %}
### Matar un servicio con kill

Para  detener se puede utilizar el comando kill -9 de esta forma:

```bash
kill -9 <PID>
```

```
pyrhos@debian:~$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
Terminado (killed)

```
{% endstep %}
{% endstepper %}
