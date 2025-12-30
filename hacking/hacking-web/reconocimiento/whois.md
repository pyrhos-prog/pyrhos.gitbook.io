# WHOIS

WHOIS es un protocolo de consulta y respuesta ampliamente utilizado, diseñado para acceder a bases de datos que almacenan información sobre recursos registrados en internet. Asociado principalmente a nombres de dominio, WHOIS también puede proporcionar detalles sobre bloques de direcciones IP y sistemas autónomos. Considérelo como una gigantesca guía telefónica de internet, que le permite consultar quién es el propietario o responsable de diversos recursos en línea.

## Who is

```shell-session
pyrhos@htb[/htb]$ whois inlanefreight.com

[...]
Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
[...]
```

Cada registro WHOIS normalmente contiene la siguiente información:

* `Domain Name`: El nombre de dominio en sí (por ejemplo, ejemplo.com)
* `Registrar`: La empresa donde se registró el dominio (por ejemplo, GoDaddy, Namecheap)
* `Registrant Contact`: La persona u organización que registró el dominio.
* `Administrative Contact`: La persona responsable de administrar el dominio.
* `Technical Contact`: La persona que maneja los problemas técnicos relacionados con el dominio.
* `Creation and Expiration Dates`: Cuándo se registró el dominio y cuándo está previsto que caduque.
* `Name Servers`: Servidores que traducen el nombre de dominio en una dirección IP.

## Historia de WHOIS

La historia de WHOIS está intrínsecamente vinculada a la visión y dedicación de [Elizabeth Feinler](https://en.wikipedia.org/wiki/Elizabeth_J._Feinler), una científica informática que jugó un papel fundamental en el desarrollo de Internet en sus inicios.

En la década de 1970, Feinler y su equipo del Centro de Información de Redes (NIC) del Instituto de Investigación de Stanford reconocieron la necesidad de un sistema para rastrear y gestionar el creciente número de recursos de red en ARPANET, precursora de la internet moderna. Su solución fue la creación del directorio WHOIS, una base de datos rudimentaria pero innovadora que almacenaba información sobre usuarios de la red, nombres de host y nombres de dominio.

## Por qué es importante WHOIS para la Web Recon

Los datos WHOIS constituyen una valiosa fuente de información para los evaluadores de penetración durante la fase de reconocimiento de una evaluación. Ofrecen información valiosa sobre la huella digital de la organización objetivo y sus posibles vulnerabilidades:

* `Identifying Key Personnel` — Los registros WHOIS suelen revelar los nombres, direcciones de correo electrónico y números de teléfono de las personas responsables de la administración del dominio. Esta información puede utilizarse para ataques de ingeniería social o para identificar posibles objetivos de campañas de phishing.
* `Discovering Network Infrastructure` — Detalles técnicos como servidores de nombres y direcciones IP ofrecen pistas sobre la infraestructura de red del objetivo. Esto puede ayudar a los evaluadores de penetración a identificar posibles puntos de entrada o configuraciones incorrectas.
* `Historical Data Analysis` — Acceder a registros históricos de WHOIS mediante servicios como [WhoisFreaks](https://whoisfreaks.com/) puede revelar cambios en la propiedad, la información de contacto o detalles técnicos a lo largo del tiempo. Esto puede ser útil para rastrear la evolución de la presencia digital del objetivo.
