---
layout:
  width: default
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

# Introducción

***

El reconocimiento web es la base de una evaluación de seguridad exhaustiva.&#x20;

Implica la recopilación sistemática de información sobre un sitio web o aplicación web.&#x20;

Constituye una parte fundamental de la recopilación de información.

<div data-with-frame="true"><img src="../.gitbook/assets/PT process.png" alt=""></div>

### Los objetivos principales del reconocimiento web

|                                  |                                                     |                                                                                                                   |
| -------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| identificación de archivos       | Localizar componentes públicos del objetivo         | <ul><li>Sitios web </li><li>Subdominios</li><li>IPs </li><li>Tecnologias</li></ul>                                |
| Detección de información oculta  | Buscar datos sensibles expuestos por error          | <ul><li>Archivos de respaldo</li><li>Configuraciones</li></ul>                                                    |
| Análisis de superficie de ataque | Identificar vulnerabilidades explotables            | <ul><li>Evaluar teconologías</li><li>Evaluar configuraciones</li><li>Evaluar puntes débiles</li></ul>             |
| Recolección de inteligencia      | Obtener información relevante para futuras acciones | <ul><li>Claves personales</li><li>Correos electrónicos</li><li>Patrones de comportamiento aprovechables</li></ul> |

Esta información se utiliza para adaptar sus ataques, lo que permite atacar debilidades específicas y eludir las medidas de seguridad.

### Tipos de reconocimiento

El reconocimiento web abarca dos metodologías fundamentales:&#x20;

* El reconocimiento web activo&#x20;
* El reconocimiento web  pasivo

Cada enfoque ofrece ventajas y desafíos distintos, y comprender sus diferencias es crucial para una recopilación de información.

### Reconocimiento activo

En el reconocimiento activo, el atacante `Interactua directamente con el objetivo` para recopilar información. Esta interacción puede adoptar diversas formas:

<table data-full-width="false"><thead><tr><th>Técnica</th><th>Descripción</th><th>Herramientas</th><th>Riesgo de detección</th></tr></thead><tbody><tr><td><code>Port Scanning</code></td><td>Identificar puertos abiertos y servicios que se ejecutan en el objetivo.</td><td><ul><li>Nmap</li><li>Masscan</li><li>Unicornscan</li></ul></td><td><strong>Alto</strong>: La interacción directa con el objetivo puede activar los sistemas de detección de intrusiones (IDS) y firewalls.</td></tr><tr><td><code>Vulnerability Scanning</code></td><td>Sondear el objetivo en busca de vulnerabilidades conocidas, como software desactualizado o configuraciones incorrectas.</td><td><ul><li>Nessus</li><li>OpenVAS</li><li>Nikto</li></ul></td><td><strong>Alto</strong>: Los escáneres envían cargas útiles de explotación que las soluciones de seguridad pueden detectar.</td></tr><tr><td><code>Network Mapping</code></td><td>Mapeo de la topología de red del objetivo, incluidos los dispositivos conectados y sus relaciones.</td><td><ul><li>Traceroute</li><li>Nmap</li></ul></td><td><strong>Medio a alto</strong>: El tráfico de red excesivo o inusual puede generar sospechas.</td></tr><tr><td><code>Banner Grabbing</code></td><td>Recuperar información de los banners mostrados por los servicios que se ejecutan en el objetivo.</td><td><ul><li>Netcat</li><li>Curl</li></ul></td><td><strong>Bajo</strong>: la captura de banners generalmente implica una interacción mínima, pero aún así se puede registrar.</td></tr><tr><td><code>OS Fingerprinting</code></td><td>Identificar el sistema operativo que se ejecuta en el objetivo.</td><td><ul><li>Nmap</li><li>Xprobe2</li></ul></td><td><strong>Bajo</strong>: la toma de huellas dactilares del sistema operativo suele ser pasiva, pero se pueden detectar algunas técnicas avanzadas.</td></tr><tr><td><code>Service Enumeration</code></td><td>Determinar las versiones específicas de los servicios que se ejecutan en puertos abiertos.</td><td><ul><li>Nmap</li></ul></td><td><strong>Bajo</strong>: similar a la captura de banners, la enumeración de servicios se puede registrar, pero es menos probable que active alertas.</td></tr><tr><td><code>Web Spidering</code></td><td>Rastrear el sitio web de destino para identificar páginas web, directorios y archivos.</td><td><ul><li>Burp Suite Spider</li><li>OWASP ZAP Spider</li><li>Scrapy</li></ul></td><td><strong>Bajo a medio</strong>: se puede detectar si el comportamiento del rastreador no está configurado cuidadosamente para imitar el tráfico legítimo.</td></tr></tbody></table>

El reconocimiento activo proporciona una visión directa y, a menudo, más completa de la infraestructura y la seguridad del objetivo. Sin embargo, también conlleva un mayor riesgo de detección, ya que las interacciones con el objetivo pueden activar alertas o levantar sospechas.

### Reconocimiento pasivo

El reconocimiento pasivo implica recopilar información sobre el objetivo sin interacción directa. Esto se basa en el análisis de información y recursos públicos, como:

<table data-full-width="false"><thead><tr><th>Técnica</th><th>Descripción</th><th>Herramientas</th></tr></thead><tbody><tr><td><code>Motores de búsqueda</code></td><td>Utilizar motores de búsqueda para descubrir información sobre el objetivo, incluidos sitios web, perfiles de redes sociales y artículos de noticias.</td><td><ul><li>Google, DuckDuckGo, Bing </li><li>Motores de búsqueda especializados (Shodan, Censys, Fofa)</li></ul></td></tr><tr><td><code>WHOIS Lookups</code></td><td>Consultar bases de datos WHOIS para recuperar detalles de registro de dominio.</td><td><ul><li>comando whois</li><li>servicios de búsqueda WHOIS en línea</li></ul></td></tr><tr><td><code>DNS</code></td><td>Análisis de registros DNS para identificar subdominios, servidores de correo y otra infraestructura.</td><td><ul><li>dig</li><li>nslookup</li><li>host </li><li>dnsenum</li><li>dnsrecon</li></ul></td></tr><tr><td><code>Analisis de archivos históricos</code></td><td>Examinar instantáneas históricas del sitio web del objetivo para identificar cambios, vulnerabilidades o información oculta.</td><td><ul><li>Wayback Machine</li></ul></td></tr><tr><td><code>Análisis de redes sociales</code></td><td>Recopilación de información de plataformas de redes sociales.</td><td><ul><li>LinkedIn</li><li>Twitter</li><li>Facebook</li><li>herramientas OSINT especializadas</li></ul></td></tr><tr><td><code>Repositorios de código</code></td><td>Analizar repositorios de código de acceso público como GitHub en busca de credenciales expuestas o vulnerabilidades.</td><td><ul><li>GitHub</li><li>GitLab</li></ul></td></tr></tbody></table>

El reconocimiento pasivo es más sigiloso y menos propenso a activar alarmas que el reconocimiento activo. Sin embargo, puede proporcionar información menos completa, ya que se basa en información ya pública.
