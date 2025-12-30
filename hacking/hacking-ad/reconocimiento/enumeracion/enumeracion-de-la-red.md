# Enumeración de la red

{% stepper %}
{% step %}
### Conocer adaptadores de red y las IPs

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Conocer la dirección mac y más información

```bash
ipconfig /all
```
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### Conocer enrutamiento direcciones IP

```bash
route print
```
{% endstep %}

{% step %}
<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

### Conocer equipos conectados a nuestra red

```bash
arp -a
```

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Conocer puertos abiertos

```bash
netstat -ano
```

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}
