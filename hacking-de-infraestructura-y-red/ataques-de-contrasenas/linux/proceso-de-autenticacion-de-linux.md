# Proceso de Autenticación de Linux

Linux gestiona la autenticación mediante una arquitectura modular basada en PAM (Pluggable Authentication Modules). Esto significa que cualquier aplicación puede delegar la autenticación a PAM, que a su vez encadena módulos configurables. Entender dónde se almacenan los hashes y cómo se validan es el punto de partida para atacar credenciales en Linux.

### /etc/passwd y /etc/shadow

Históricamente, las contraseñas se guardaban en `/etc/passwd` junto con los demás datos del usuario. Como ese archivo es legible por todos los usuarios, se separó la parte sensible a `/etc/shadow`, que solo root puede leer.

#### /etc/passwd

```
usuario:x:1001:1001:Nombre Apellido:/home/usuario:/bin/bash
```

| Campo             | Descripción                                                                |
| ----------------- | -------------------------------------------------------------------------- |
| `usuario`         | Nombre de usuario                                                          |
| `x`               | Indica que el hash está en `/etc/shadow` (si hay hash aquí, está obsoleto) |
| `1001`            | UID                                                                        |
| `1001`            | GID                                                                        |
| `Nombre Apellido` | GECOS — info descriptiva                                                   |
| `/home/usuario`   | Directorio home                                                            |
| `/bin/bash`       | Shell de login                                                             |

#### /etc/shadow

```
usuario:$6$salt$hash:19000:0:99999:7:::
```

| Campo          | Descripción                          |
| -------------- | ------------------------------------ |
| `usuario`      | Nombre de usuario                    |
| `$6$salt$hash` | Hash de la contraseña                |
| `19000`        | Días desde epoch hasta último cambio |
| `0`            | Días mínimos entre cambios           |
| `99999`        | Días máximos de validez              |
| `7`            | Días de aviso antes de expirar       |

#### Formatos de hash en /etc/shadow

| Prefijo         | Algoritmo                                                |
| --------------- | -------------------------------------------------------- |
| `$1$`           | MD5crypt                                                 |
| `$2y$` / `$2b$` | bcrypt                                                   |
| `$5$`           | SHA-256crypt                                             |
| `$6$`           | SHA-512crypt (estándar moderno en la mayoría de distros) |
| `$y$`           | yescrypt (Fedora, RHEL 9+, Ubuntu 22.04+)                |

### PAM — Pluggable Authentication Modules

PAM es el framework que conecta aplicaciones (login, sshd, sudo, su) con los mecanismos de autenticación. La configuración está en `/etc/pam.d/`:

```
/etc/pam.d/sshd          # autenticación SSH
/etc/pam.d/sudo          # autenticación sudo
/etc/pam.d/login         # login de consola
/etc/pam.d/common-auth   # política común (Debian/Ubuntu)
```

Cada archivo define una cadena de módulos con directivas `required`, `sufficient`, `optional`:

```
auth   required   pam_unix.so    # valida contra /etc/shadow
auth   optional   pam_sss.so     # valida contra SSSD (LDAP/AD si está configurado)
```

Los módulos relevantes para un atacante son:

| Módulo                              | Función                                       |
| ----------------------------------- | --------------------------------------------- |
| `pam_unix.so`                       | Autenticación local con `/etc/shadow`         |
| `pam_sss.so`                        | Integración con SSSD (LDAP, Active Directory) |
| `pam_ldap.so`                       | LDAP directo                                  |
| `pam_exec.so`                       | Ejecuta script externo — potencial backdoor   |
| `pam_tally2.so` / `pam_faillock.so` | Control de lockout                            |

> Si se tiene acceso de escritura a `/etc/pam.d/`, insertar `pam_exec.so` con un script que loguee credenciales es una técnica de persistencia efectiva aunque ruidosa.

### Flujo de autenticación SSH

```
Cliente SSH → sshd → PAM → pam_unix.so → /etc/shadow
                                ↓
                    (si SSSD configurado) → pam_sss.so → LDAP/AD
```

Para autenticación por clave pública, PAM no interviene: sshd valida directamente la clave contra `~/.ssh/authorized_keys`.

### nsswitch.conf — resolución de nombres

`/etc/nsswitch.conf` define el orden de fuentes para resolver usuarios, grupos y otros:

```
passwd:   files sss     # primero /etc/passwd, luego SSSD
shadow:   files sss
group:    files sss
```

En sistemas integrados con Active Directory (via SSSD o winbind), los usuarios del dominio son resolubles localmente, lo que amplía la superficie de credenciales accesibles.

### Extracción de hashes (requiere root)

```bash
cat /etc/shadow
# o directamente para cracking:
unshadow /etc/passwd /etc/shadow > hashes_para_john.txt
john hashes_para_john.txt --wordlist=rockyou.txt
hashcat -m 1800 shadow_hashes.txt rockyou.txt    # SHA-512crypt
hashcat -m 7400 shadow_hashes.txt rockyou.txt    # SHA-256crypt
```

> En sistemas modernos con yescrypt (`$y$`), Hashcat usa `-m 160`. El cracking es significativamente más lento que SHA-512crypt por el diseño memory-hard del algoritmo.
