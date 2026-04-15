# La red detrás del pivoting

El pivoting ocurre cuando un equipo comprometido actúa como “puente” hacia otras redes internas.\
En la práctica, el objetivo no es entender la red en teoría, sino identificar **desde qué interfaces, rutas y puertos puedes moverte a otros segmentos**.

### Direccionamiento IP y NIC

#### Qué miras en un sistema comprometido

En Linux:

```
ip a
```

Ejemplo típico:

* eth0 → 192.168.1.50 (red externa o DMZ)
* eth1 → 10.10.10.5 (red interna)
* tun0 → 10.8.0.2 (VPN)

#### Interpretación práctica

Si ves esto:

* 192.168.1.50 → red “visible” desde fuera o semi pública
* 10.10.10.5 → red interna privada

**Conclusión**: el host ya está dentro de dos redes → **es candidato directo a pivot**

#### Ejemplo real de pivoting

Supón:

* Tu máquina atacante: 192.168.0.10
* Máquina comprometida: 192.168.0.50 + 10.0.0.5

Acciones típicas:

* Desde 192.168.0.50 haces escaneo a 10.0.0.0/24
* Descubres un servidor interno no expuesto

### Interfaces de red

#### Linux

* lo → localhost (irrelevante para pivoting)
* eth0 / eth1 → redes físicas o virtuales
* tun0 → VPN (muy común en labs y entornos reales)

#### Caso práctico

Si tienes:

```
eth0 → Internet / red externa
eth1 → red interna corporativa
```

Significa:

* El sistema puede actuar como router manual
* Puedes encaminar tráfico hacia redes que tu máquina atacante no ve

### IP pública vs privada

#### Ejemplo típico

* IP pública: 80.120.x.x → servidor expuesto
* IP privada: 10.x.x.x / 192.168.x.x → red interna

#### Situación real

Si comprometes un servidor con IP pública:

* Solo ves ese host desde Internet
* Pero desde dentro puedes ver:

```
10.10.0.0/24 (base de datos)
10.10.1.0/24 (AD)
```

Pivoting = explotar ese salto hacia redes privadas

### Gateway y salida

Ver gateway en Linux:

```
ip route
```

Ejemplo:

```
default via 192.168.1.1 dev eth0
10.10.0.0/24 via 10.10.0.1 dev eth1
```

#### Interpretación directa:

* Todo lo desconocido sale por 192.168.1.1
* La red 10.10.0.0/24 tiene ruta específica

Si puedes modificar rutas o usar el host como proxy:

* puedes alcanzar redes internas sin acceso directo

### Máscara de subred

Ejemplo:

* 192.168.1.0/24 → red local
* 10.10.10.0/24 → red interna

#### Qué implica en la práctica:

Si estás en:

```
192.168.1.50/24
```

puedes hablar directamente con:

```
192.168.1.1 – 192.168.1.254
```

Pero NO con:

```
10.10.10.0/24
```

A menos que el host comprometido actúe como puente

### Enrutamiento

Ver rutas:

```
ip route
```

#### Caso práctico:

Antes del pivot:

```
solo Internet
```

Después de comprometer host:

```
192.168.1.0/24 directo
10.10.10.0/24 vía 192.168.1.50
```

Esto es literalmente pivoting a nivel de red

#### Ejemplo mental claro

Tu máquina:

* no ve 10.10.10.0/24

Host comprometido:

* sí lo ve

tú usas ese host como “router manual”

### Puertos y servicios

Escaneo típico:

```
nmap 10.10.10.5
```

Ejemplo resultado:

```
22/tcp open ssh
80/tcp open http
3306/tcp open mysql
```

#### Interpretación:

* 22 → posible acceso lateral
* 80 → panel web interno
* 3306 → base de datos interna crítica

En pivoting, estos puertos no estaban accesibles desde fuera, solo desde la red interna.

### Ejemplo completo de escenario real

> * Comprometes servidor web:
>   * 192.168.1.50
> * Descubres otra interfaz:
>   * 10.10.10.5
> * Escaneas red interna desde ese host:
>   * Encuentras DC (Domain Controller) en 10.10.10.10
> * Usas el host como pivot:
>   * accedes a servicios internos no expuestos

La identificación de estas propiedades en hosts comprometidos permite determinar posibles rutas de acceso hacia redes no directamente alcanzables.
