# Enumeración de puertos y servicios

Después de tener el mapa de la red, hay que enumerar los puertos y los servicios del host para buscar posibles vectores de ataque.

### Enumeración de puertos

```
sudo nmap -p- --open --min-rate 5000 -vvv -sS -n -Pn [ip] -oN archivo.txt
```

### Enumeración de los servicios

```
sudo nmap -p[puertos] -sCV  [ip] -oN archivo.txt
```
