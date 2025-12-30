# Puerto 25 (SMTP)

### Enumeración con telnet

Podemos descubrir la versión y usuario del servidor SMTP usando telnet.

```
telnet <ip-víctima> 25

```

### Enumeración con metasploit

Con este módulo de metasploit podemos enumerar el usuario en smtp

```
use auxiliar/scanner/smtp/smtp_enum
msf auxiliar(smtp_enum) > set rhosts 192.168.1.107
msf auxiliar(smtp_enum) > set rport 25
msf auxiliar(smtp_enum) > set USER_FILE <ruta al diccionario>
msf auxiliar(smtp_enum) > exploit
```

