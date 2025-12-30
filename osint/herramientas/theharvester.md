---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/BGEOiV6kHOgZlzy7weRR/theharvester
---

# TheHarvester

Es una herramienta que busca información pública de dominios a través de motores de busqueda como Google, Bing, Yahoo...

Podemos hacer una búsqueda de la siguiente manera:

```bash
theHarvester -d linkedin.com -b bing -l 100
```

{% stepper %}
{% step %}
### Parámetro: -d

Este parámetro especifica el dominio objetivo del cual se buscará la información
{% endstep %}

{% step %}
### Parámetro: -b bing

Este parámetro especifica el motor de búsqueda que TheHarvester utilizará para realizar la búsqueda de información. TheHarvester puede utilizar una amplia gama de motores de búsqueda como:&#x20;

* Google
* Bing
* PGP
* LinkedIn

&#x20;Cada motor de búsqueda tiene su propia base de datos y métodos de búsqueda.
{% endstep %}

{% step %}
### Parámetro: -l 100

Este parámetro limita los resultados a 100.
{% endstep %}
{% endstepper %}
