---
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
---

# Introducción a las redes

Una red informática es un conjunto de equipos interconectados que comparten información, recursos y servicios. Se compone de una parte física (hardware, cables, routers, switches) y una parte lógica (software, protocolos como TCP/IP).

## Componentes de una Red

<table><thead><tr><th width="184.33333333333331">Nombre</th><th>Definición</th><th data-hidden data-type="files"></th><th data-hidden></th><th data-hidden data-type="content-ref"></th></tr></thead><tbody><tr><td><i class="fa-share-nodes">:share-nodes:</i> <strong>Nodo</strong></td><td><strong>Un nodo es cualquier dispositivo conectado a una red, que pueda recibir y enviar datos</strong></td><td></td><td></td><td></td></tr><tr><td><p><i class="fa-signal-stream">:signal-stream:</i> <strong>Medio de</strong></p><p><strong>transmisión</strong></p></td><td><strong>Es el medio por el que se transmiten los datos, puede ser físico (Cable de cobre, fibra óptica) o inalámbrico (Wireless).</strong></td><td></td><td></td><td><a href="https://github.com/GitbookIO/gitbook-templates/blob/main/product-docs/broken-reference/README.md">https://github.com/GitbookIO/gitbook-templates/blob/main/product-docs/broken-reference/README.md</a></td></tr><tr><td><p><i class="fa-router">:router:</i> <strong>Dispositivos</strong></p><p><strong>de red</strong></p></td><td><strong>Equipos como routers, switchers, y puntos de acceso que facilitan el tráfico de datos.</strong></td><td></td><td></td><td></td></tr><tr><td><i class="fa-scale-balanced">:scale-balanced:</i> <strong>Protocolos</strong></td><td><strong>Son el conjunto de reglas que definen cómo se comunican los dispositivos en la red ( TCP/IP/FTP/SSH/HTTP... )</strong></td><td></td><td></td><td></td></tr></tbody></table>

## Conceptos básicos

<table><thead><tr><th width="184.33333333333331">Nombre</th><th>Definición</th><th data-hidden data-type="files"></th><th data-hidden></th><th data-hidden data-type="content-ref"></th></tr></thead><tbody><tr><td><i class="fa-share-nodes">:share-nodes:</i> <strong>Cliente</strong></td><td><strong>Dispositivo que se conecta a un servidor para utilizar un servicio.</strong></td><td></td><td></td><td></td></tr><tr><td><i class="fa-signal-stream">:signal-stream:</i> <strong>Servidor</strong></td><td><strong>Proporciona un servicio a otros dispositivos</strong></td><td></td><td></td><td><a href="https://github.com/GitbookIO/gitbook-templates/blob/main/product-docs/broken-reference/README.md">https://github.com/GitbookIO/gitbook-templates/blob/main/product-docs/broken-reference/README.md</a></td></tr><tr><td><i class="fa-router">:router:</i> <strong>IP</strong></td><td><strong>Es el identificador del dispositivo conectado a una red, puede ser pública o privada.</strong></td><td></td><td></td><td></td></tr><tr><td><i class="fa-scale-balanced">:scale-balanced:</i> <strong>Puerto</strong></td><td><strong>Es un número que identifica una aplicación o servicio específico (21,22,80,443...)</strong></td><td></td><td></td><td></td></tr><tr><td><i class="fa-scale-balanced">:scale-balanced:</i> <strong>Protocolo</strong></td><td><strong>Reglas y estándares para la comunicación en una red.</strong></td><td></td><td></td><td></td></tr></tbody></table>

## Diagrama básico de una red

<figure><img src="https://1680330859-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2Fz0iyBve0fcLSJpEvb9YL%2Fuploads%2FTzzrQr8kwEXN1SRbuIH9%2Fimage.png?alt=media&#x26;token=b7520a50-1593-43db-bfc7-cc4176f2a27f" alt=""><figcaption></figcaption></figure>

Las direcciones IP están compuestas por 32 bits de 4 octetos cada uno. Cada clase de una dirección de red determina:

* Una máscara por defecto
* Un rango IP
* Cantidad de redes y de hosts por red

## Dirección clase A, B, C, D y E

<table><thead><tr><th align="center">CLASE</th><th align="center">RANGO</th><th align="center">Máscara por defecto</th><th width="128" align="center">Cantidad de redes</th><th align="center">Cantidad de Hosts</th><th align="center">Aplicación</th></tr></thead><tbody><tr><td align="center"><strong>A</strong></td><td align="center">1 - 126</td><td align="center">255.0.0.0</td><td align="center">128</td><td align="center">16,777,214</td><td align="center">Redes muy grandes</td></tr><tr><td align="center"><strong>B</strong></td><td align="center">128 - 191</td><td align="center">255.255.0.0</td><td align="center">16,384</td><td align="center">65,534</td><td align="center">Redes medianas</td></tr><tr><td align="center"><strong>C</strong></td><td align="center">192 - 223</td><td align="center">255.255.255.0</td><td align="center">2,097,152</td><td align="center">254</td><td align="center">Redes locales (LAN)</td></tr><tr><td align="center"><strong>D</strong></td><td align="center">224 - 239</td><td align="center">No definida</td><td align="center">N/A</td><td align="center">N/A</td><td align="center">Multicast</td></tr><tr><td align="center"><strong>E</strong></td><td align="center">240 - 255</td><td align="center">No definida</td><td align="center">N/A</td><td align="center">N/A</td><td align="center">Investigación</td></tr></tbody></table>

### Clase A

* **Dirección de red:** Primer octeto (8 bits). El primer bit siempre es `0`.
* **Dirección de hosts:** Últimos 3 octetos (24 bits).

### Clase B

* **Dirección de red:** Primeros 2 octetos (16 bits). Los primeros dos bits son `10`.
* **Dirección de hosts:** Últimos 2 octetos (16 bits).

### Clase C

* **Dirección de red:** Primeros 3 octetos (24 bits). Los primeros tres bits son `110`.
* **Dirección de hosts:** Último octeto (8 bits).

**Tabla de ejemplos por clase (Binario vs Decimal)**

| Clase | Máscara en Binario                    | Máscara Decimal | Prefijo (CIDR) |
| ----- | ------------------------------------- | --------------- | :------------: |
| **A** | `11111111.00000000.00000000.00000000` | 255.0.0.0       |       /8       |
| **B** | `11111111.11111111.00000000.00000000` | 255.255.0.0     |       /16      |
| **C** | `11111111.11111111.11111111.00000000` | 255.255.255.0   |       /24      |

## Máscara de red

Se utiliza para diferenciar la parte de red de la parte de host. Los bits en `1` representan la red y los bits en `0` representan los hosts.

1. **Porción de red:** Representa los bits encendidos (identificación del segmento).
2. **Porción de hosts:** Representa los bits apagados (identificación del dispositivo).
