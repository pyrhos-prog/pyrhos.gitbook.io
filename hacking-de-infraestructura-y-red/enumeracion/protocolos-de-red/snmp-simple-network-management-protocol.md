# SNMP — Simple Network Management Protocol

SNMP (Simple Network Management Protocol) es el protocolo estándar para la gestión y monitorización de dispositivos de red: routers, switches, servidores, impresoras, firewalls... Opera en el puerto **161** (UDP) para consultas y **162** (UDP) para traps (notificaciones enviadas por el dispositivo).

A pesar de su nombre, SNMP es una fuente de información extremadamente rica para el reconocimiento — puede revelar configuraciones del sistema, interfaces de red, procesos en ejecución, rutas, usuarios y mucho más.

### Versiones de SNMP

| Versión     | Autenticación        | Cifrado      | Notas                              |
| ----------- | -------------------- | ------------ | ---------------------------------- |
| **SNMPv1**  | Community string     | No           | Más antiguo, muy inseguro          |
| **SNMPv2c** | Community string     | No           | Más eficiente, igualmente inseguro |
| **SNMPv3**  | Usuario + contraseña | Sí (AES/DES) | Versión segura, la recomendada     |

Las versiones 1 y 2c usan **community strings** como único mecanismo de autenticación. Son básicamente contraseñas en texto claro que viajan en cada petición SNMP. Los valores por defecto son `public` (lectura) y `private` (escritura).

### MIB — Management Information Base

La MIB es la estructura jerárquica de objetos que se pueden consultar via SNMP. Cada objeto tiene un **OID** (Object Identifier) único. Algunos OIDs relevantes:

| OID                     | Información                                     |
| ----------------------- | ----------------------------------------------- |
| `1.3.6.1.2.1.1.1.0`     | Descripción del sistema (sysDescr)              |
| `1.3.6.1.2.1.1.3.0`     | Uptime del sistema                              |
| `1.3.6.1.2.1.1.5.0`     | Hostname del sistema                            |
| `1.3.6.1.2.1.2.2`       | Interfaces de red                               |
| `1.3.6.1.2.1.4.20`      | Tabla de IPs del dispositivo                    |
| `1.3.6.1.2.1.4.21`      | Tabla de rutas                                  |
| `1.3.6.1.2.1.6.13`      | Tabla de conexiones TCP                         |
| `1.3.6.1.2.1.25.1`      | Información del host (procesos, memoria, disco) |
| `1.3.6.1.2.1.25.4.2`    | Procesos en ejecución                           |
| `1.3.6.1.2.1.25.6.3`    | Software instalado                              |
| `1.3.6.1.4.1.77.1.2.25` | Cuentas de usuario (Windows)                    |

### Enumeración

#### Nmap

```bash
# Detección del servicio SNMP
nmap -sU -p 161 target
nmap -sU -sV -p 161 target

# Scripts de nmap para SNMP
nmap -sU -p 161 --script snmp-info,snmp-sysdescr target
nmap -sU -p 161 --script snmp-brute target                   # brute force de community strings
nmap -sU -p 161 --script snmp-interfaces target              # interfaces de red
nmap -sU -p 161 --script snmp-processes target               # procesos en ejecución
nmap -sU -p 161 --script snmp-netstat target                 # conexiones activas
nmap -sU -p 161 --script snmp-win32-users target             # usuarios Windows
```

#### snmpwalk — recorrer toda la MIB

`snmpwalk` consulta el dispositivo y devuelve todos los valores disponibles para una community string dada.

```bash
# Recorrer toda la MIB con community string "public"
snmpwalk -v2c -c public target

# Recorrer desde un OID específico
snmpwalk -v2c -c public target 1.3.6.1.2.1.1        # info del sistema
snmpwalk -v2c -c public target 1.3.6.1.2.1.25.4.2   # procesos
snmpwalk -v2c -c public target 1.3.6.1.2.1.4.20      # IPs

# Con SNMPv1
snmpwalk -v1 -c public target

# Con SNMPv3 (autenticación)
snmpwalk -v3 -l authPriv -u usuario -a SHA -A contraseña -x AES -X privpassword target
```

#### snmpget — consulta de OID específico

```bash
# Obtener un valor específico por OID
snmpget -v2c -c public target 1.3.6.1.2.1.1.1.0    # descripción del sistema
snmpget -v2c -c public target 1.3.6.1.2.1.1.5.0    # hostname
```

#### onesixtyone — brute force de community strings

`onesixtyone` es una herramienta rápida para descubrir community strings válidas:

```bash
# Wordlist básica de community strings comunes
echo "public
private
community
manager
monitor
admin
cisco
snmpd" > community_strings.txt

# Probar en un target
onesixtyone -c community_strings.txt target

# Probar en un rango de red
onesixtyone -c community_strings.txt -i targets.txt

# SecLists tiene una wordlist específica:
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt target
```

#### snmp-check — enumeración detallada

```bash
# Enumeración completa del sistema
snmp-check target
snmp-check -c public target
snmp-check -v 2c -c public target

# Muestra: info del sistema, interfaces, rutas, procesos,
#          servicios, cuentas de usuario, software instalado
```

***

### Riesgos y misconfiguraciones

| Riesgo                                                 | Descripción                                                                                                                                      |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Community strings por defecto**                      | `public` y `private` son los valores por defecto. Si no se cambian, cualquiera puede leer (y a veces escribir) la configuración del dispositivo. |
| **SNMPv1/v2c sin cifrado**                             | Las community strings viajan en texto claro. Capturables con Wireshark en una red local.                                                         |
| **Community string "private" con acceso de escritura** | Permite modificar la configuración del dispositivo (cambiar rutas, deshabilitar interfaces, etc.).                                               |
| **Exposición al exterior**                             | SNMP accesible desde internet expone información detallada de la infraestructura.                                                                |
| **Enumeración de usuarios (Windows)**                  | En sistemas Windows con SNMP activo, el OID de usuarios revela las cuentas locales del sistema.                                                  |
| **Enumeración de software**                            | La lista de software instalado revela versiones con posibles CVEs.                                                                               |
| **Procesos que revelan contraseñas**                   | Los procesos en ejecución a veces incluyen argumentos con credenciales (ej: scripts que reciben contraseñas como argumentos).                    |

> Si `snmpwalk` con la community string `public` devuelve resultados, en pocos segundos se puede obtener el hostname, IPs de todas las interfaces, rutas de red, procesos activos, software instalado y cuentas de usuario — todo sin autenticación real.

> En dispositivos de red (routers Cisco, switches...) con SNMP de escritura activo y community string `private` por defecto, es posible **modificar la configuración del dispositivo** directamente via SNMP — cambiar rutas, deshabilitar puertos o incluso hacer downgrade del firmware.
