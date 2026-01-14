# Enumeración de puertos y servicios

Una vez tengamos identificado un host activo dentro de la red, el siguiente paso es enumerar sus puertos y servicios. Esto nos permite identificar puntos de entrada y posibles vectores de ataque.

### Enumeración de puertos

```
sudo nmap -p- --open --min-rate 5000 -vvv -sS -n -Pn [ip] -oN archivo.txt
```

| `-p-`             | Escanea todos los puertos (1–65535) |
| ----------------- | ----------------------------------- |
| `--open`          | Muestra solo puertos abiertos       |
| `--min-rate 5000` | Aumenta la velocidad del escaneo    |
| `-sS`             | Escaneo SYN (rápido y sigiloso)     |
| `-n`              | Sin resolución DNS                  |
| `-Pn`             | No realiza ping previo              |
| `-vvv`            | Salida detallada                    |
| `-oN`             | Guarda el resultado en texto        |

### Enumeración de los servicios

```
sudo nmap -p[puertos] -sCV  [ip] -oN archivo.txt
```

| `-p`  | Puertos detectados previamente |
| ----- | ------------------------------ |
| `-sC` | Scripts por defecto de Nmap    |
| `-sV` | Detección de versiones         |
| `-oN` | Guarda el resultado            |

### Enumeración UDP&#x20;

```
sudo nmap -sU -p --top-ports 100 [IP]
```

