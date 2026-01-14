# Enumeración del sistema operativo

### Enumeración con Nmap

Nmap utiliza huellas TCP/IP para inferir el sistema operativo.

```
sudo nmap -0 [ip]
```

#### Parametros para enumeracion del S.O

| Parámetro        | Función                            |
| ---------------- | ---------------------------------- |
| `-O`             | Detección de sistema operativo     |
| `--osscan-limit` | Limita el escaneo a hosts viables  |
| `--osscan-guess` | Fuerza una estimación más agresiva |

**Ejemplo**

```
sudo nmap -O --osscan-guess [IP]
```

