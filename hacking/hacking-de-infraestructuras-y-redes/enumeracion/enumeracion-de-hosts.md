# Enumeración de hosts

El primer paso al enumerar una red es mapear los dispositivos de la red para saber como esta estructurada. Para ello se usan herramienta de escaneo como nmap, masscan, unicornscan entre otras

### Enumeración de hosts con ICMP

#### Nmap

```bash
sudo nmap -PE -PM -PP -sn 192.168.1.1/24
```

* `-PE:` Envía un paquete ICMP tipo 8 (el clásico "ping") a la dirección IP objetivo. Es el método más común para ver si un host responde, aunque muchos firewalls modernos bloquean este tipo de paquetes.
* `-PP` : Envía un paquete ICMP tipo 13. Se utiliza como alternativa al ping convencional. A veces, los firewalls que bloquean peticiones de "Echo" (`-PE`) olvidan bloquear las peticiones de marca de tiempo.
* `-PM` : Envía un paquete ICMP tipo 17. Intenta obtener la máscara de subred del objetivo. sirve para intentar evadir filtros que solo bloquean el ICMP estándar.
* `-sn`: Antiguamente conocido como `-sP`. Este parámetro le dice a Nmap que no realice un escaneo de puertos después del descubrimiento.
* `/24`: define el rango de la red en el que nmap va a buscar dispositivos.

#### Fping

```
fping -g 199.66.11.0/24
```

#### Netdiscover

```
sudo netdiscover -i wlan0
```

### Enumeración de hosts con ARP

#### Arp-scan

```
arp-scan --localnet
```
