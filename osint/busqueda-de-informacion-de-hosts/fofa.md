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
---

# FOFA

#### Fofa es una plataforma que sirve para buscar información de Hosts

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### **Reglas Generales**

> * Si el _keyword_ no lleva sintaxis ni filtro → FOFA buscará en **HTML**, **cabeceras HTTP** y **URL**.
> * Para varias condiciones AND/OR → **usar paréntesis `()`** para definir prioridad.

### Operadores Disponibles

| Operador | Descripción                                                                        |
| -------- | ---------------------------------------------------------------------------------- |
| `=`      | Igual — si el campo **no existe o está vacío, también lo muestra.**                |
| `==`     | Igual exacto — el campo **debe existir** (o estar vacío si se usa explícitamente). |
| `!=`     | Distinto — solo cuando el campo **existe**.                                        |
| `*=`     | Búsqueda difusa (usa `*` o `?`).                                                   |
| `&&`     | AND                                                                                |
| \`       |                                                                                    |
| `()`     | Control de prioridad                                                               |

## **Cheat Sheet**

{% tabs %}
{% tab title="Operadores" %}
**Operadores:**

* `=` igual
* `==` igual estricto
* `!=` distinto
* `*=` difuso (\*, ?)
* `&&` AND
* `||` OR
* `()` prioridad
{% endtab %}

{% tab title="General" %}
**General:**

* `ip=` IPv4/IPv6/segmento
* `port=` puerto
* `domain=` dominio
* `host=` hostname
* `os=` sistema operativo
* `server=` servidor
* `asn=` AS number
* `org=` organización
* `is_domain=true/false`
* `is_ipv6=true/false`
{% endtab %}

{% tab title="Special Labels" %}
**Etiquetas Especiales:**

* `app=` fingerprint FOFA
* `fid=` FeatureID
* `product=` producto
* `product.version=` versión
* `category=` categoría
* `type=service/subdomain`
* `cloud_name=` proveedor
* `is_cloud=true/false`
* `is_fraud=true/false`
* `is_honeypot=true/false`
{% endtab %}

{% tab title="Servicio" %}
**Servicios (type=service):**

* `protocol=` protocolo
* `banner=` banner
* `banner_hash=` hash
* `banner_fid=` fingerprint estructural
* `base_protocol=tcp/udp`
{% endtab %}

{% tab title="Web" %}
**Web (type=subdomain):**

* `title=` título
* `header=` cabecera
* `header_hash=` hash
* `body=` HTML
* `body_hash=` hash
* `js_name=` script
* `js_md5=` hash script
* `cname=` CNAME
* `cname_domain=` dominio raíz
* `icon_hash=` favicon
* `status_code=` HTTP
* `icp=` licencia ICP
* `sdk_hash=` SDK hash
{% endtab %}

{% tab title="Geo" %}
**Ubicación:**

* `country=` país
* `region=` región
* `city=` ciudad
{% endtab %}

{% tab title="Certificados" %}
**Certificados:**

* `cert=` búsqueda general
* `cert.subject=` sujeto
* `cert.issuer=` emisor
* `cert.subject.cn=` CN
* `cert.domain=` dominio
* `cert.is_equal=true/false`
* `cert.is_valid=true/false`
* `cert.is_match=true/false`
* `cert.is_expired=true/false`
* `jarm=` fingerprint
* `tls.version=` versión
* `tls.ja3s=` ja3s
* `cert.sn=` serial
* `cert.not_after.before/after=`
* `cert.not_before.before/after=`
{% endtab %}

{% tab title="Fechas" %}
**Fechas:**

* `after=` actualizado después
* `before=` actualizado antes
* `after && before` rango
{% endtab %}

{% tab title="Unique IP" %}
**Unique IP (solo usarlas solas):**

* `port_size=` exacto
* `port_size_gt=` mayor
* `port_size_lt=` menor
* `ip_ports=` puertos específicos
* `ip_country=`
* `ip_region=`
* `ip_city=`
* `ip_after=`
* `ip_before=`
{% endtab %}
{% endtabs %}

{% embed url="https://en.fofa.info/" %}

