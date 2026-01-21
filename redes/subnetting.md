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
---

# Subnetting

### ¿Qué es subnetting?

El subnetting es el proceso de dividir una red IPv4 grande en subredes más pequeñas, con el objetivo de:

* Optimizar el uso de direcciones IP
* Mejorar la organización de la red
* Reducir tráfico innecesario
* Facilitar el enrutamiento y la seguridad

Una subred es un segmento lógico de red donde todas las IP comparten la misma dirección de red.

### Conceptos clave

En cualquier subred IPv4 siempre existen:

* **Network address** (identifica la subred)
* **Broadcast address(** envía tráfico a todos los hosts)
* **Primer host usable**
* **Último host usable**
* **Número total de hosts**

### Ejemplo

**IP:** `192.168.12.160`\
**Máscara:** `255.255.255.192`\
**CIDR:** `/26`

### IP y máscara en binario

#### Dirección IPv4

```
192.168.12.160
11000000.10101000.00001100.10100000
```

#### Máscara de subred

```
255.255.255.192
11111111.11111111.11111111.11000000
```

* Bits en **1** → parte de red (no cambian)
* Bits en **0** → parte de host (pueden variar)

### Separación red / host

```
IP:      11000000.10101000.00001100.10|100000
Máscara: 11111111.11111111.11111111.11|000000
```

* Parte de red: fija
* Parte de host: variable

***

### Cálculo de direcciones importantes

#### Dirección de red (host bits = 0)

```
192.168.12.128
```

#### Dirección de broadcast (host bits = 1)

```
192.168.12.191
```

***

### Rango de hosts utilizables

| Tipo        | Dirección      |
| ----------- | -------------- |
| Network     | 192.168.12.128 |
| Primer host | 192.168.12.129 |
| Último host | 192.168.12.190 |
| Broadcast   | 192.168.12.191 |

**Total de direcciones:** 64\
**Hosts utilizables:** 62 (64 − 2)

### Subnetting en subredes más pequeñas

#### Objetivo

Dividir la subred `/26` en **4 subredes**.

#### Paso 1: calcular bits necesarios

```
4 subredes = 2² → se añaden 2 bits
```

```
/26 → /28
```

Nueva máscara:

```
255.255.255.240
```

#### Paso 2: tamaño de cada subred

```
64 direcciones / 4 = 16 direcciones por subred
```

#### Resultado final

| Subred | Network        | Primer host | Último host | Broadcast | CIDR |
| ------ | -------------- | ----------- | ----------- | --------- | ---- |
| 1      | 192.168.12.128 | .129        | .142        | .143      | /28  |
| 2      | 192.168.12.144 | .145        | .158        | .159      | /28  |
| 3      | 192.168.12.160 | .161        | .174        | .175      | /28  |
| 4      | 192.168.12.176 | .177        | .190        | .191      | /28  |

### Subnetting mental

#### Identificar el octeto que cambia

| CIDR    | Octeto variable |
| ------- | --------------- |
| /8      | 1º              |
| /16     | 2º              |
| /24     | 3º              |
| /25–/32 | 4º              |

Ejemplo:

```
192.168.1.1/25     solo cambia el 4º octeto
```

#### Regla del módulo (CIDR % 8)

```
/25 % 8 = 1
```

```
Tamaño de bloque = 2^(8 - 1) = 128
```

#### Tabla rápida de bloques

| Resto | Tamaño |
| ----- | ------ |
| 0     | 256    |
| 1     | 128    |
| 2     | 64     |
| 3     | 32     |
| 4     | 16     |
| 5     | 8      |
| 6     | 4      |
| 7     | 2      |

### Ejemplo /25

```
192.168.1.0 – 127
```

* Network: `.0`
* Broadcast: `.127`
* Hosts: `.1 – .126`

```
192.168.1.128 – 255
```

* Network: `.128`
* Broadcast: `.255`
* Hosts: `.129 – .254`
