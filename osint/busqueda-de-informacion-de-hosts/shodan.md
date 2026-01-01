---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
metaLinks:
  alternates:
    - https://app.gitbook.com/s/BGEOiV6kHOgZlzy7weRR/shodan-cheat-sheet
---

# Shodan

#### **Shodan es un motor de búsqueda especializado en dispositivos conectados.**

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Cheat Sheet

{% tabs %}
{% tab title="Identidad & Red" %}
**IP / Red / Host**

* `"texto"` → búsqueda general
* `ip:1.2.3.4` → IP exacta
* `ip:10.0.0.0/8` → rango/CIDR
* `net:203.0.113.0/24` → subred
* `hostname:google.com` → hostname / FQDN
* `port:PUERTO` → puerto abierto

**Organización / ISP**

* `org:"Nombre"`
* `isp:"Proveedor"`

**Visibilidad**

* `has_screenshot:true` → dispositivos con captura
{% endtab %}

{% tab title="Software & Contenido" %}
**Software / Servicios**

* `product:nginx` → producto
* `version:"1.2.3"` → versión
* `os:windows` · `os:"linux 2.6"` → sistema operativo

#### Cámaras

* `device:webcam` → dispositivos que sean cámaras
* `title:camera`  → cámaras a través del titulo

**Web / HTML**

* `http.title:"texto"` → dentro de
* `http.html:"powered by Joomla"` → dentro del HTML

**Hash**

* `hash:-209355447` → banners/HTML exactos

**Vulnerabilidades**

* `vuln:CVE-2021-44228` (_requiere plan pago_)
{% endtab %}

{% tab title="Geografía & Tiempo" %}
**Geolocalización**

* `city:Ciudad`
* `country:PAIS` (código ISO)

**Fecha de escaneo**

* `after:01/01/2024`
* `before:12/12/2023`
* Ejemplo: `apache after:01/01/2024`
{% endtab %}
{% endtabs %}

{% embed url="https://www.shodan.io/" %}
