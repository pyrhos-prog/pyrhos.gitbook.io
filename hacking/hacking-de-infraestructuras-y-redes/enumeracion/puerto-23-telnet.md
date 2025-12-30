# Puerto 23 (TELNET)

### Enumeración con nmap

Nmap tiene varios scripts para enumerar el protocolo telnet

```
pyrhos@debian-laptop:~$ ls /usr/share/nmap/scripts/ | grep "telnet" 
telnet-brute.nse
telnet-encryption.nse
telnet-ntlm-info.nse
```

También se puede hacer de esta manera:

```
pyrhos@debian-laptop:~$ nmap -p23 --script=*telnet* <ip-víctima>
```

### Enumeración manual

Conectarse a la víctima:

```
telnet <ip-víctima> 23
```

Información del banner:

```
nc -nv 23
```
