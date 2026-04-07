---
icon: windows
---

# Obtención de credenciales

## Proceso de Autenticación de Windows

Entender cómo Windows autentica a los usuarios es la base para saber por qué SAM, LSASS y NTDS.dit son objetivos tan valiosos. El proceso implica varios componentes que interactúan entre sí y que, en cada paso, almacenan o transmiten credenciales de formas distintas.

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

### Componentes principales

**WinLogon** (`winlogon.exe`) gestiona la sesión de inicio de sesión interactivo: captura las credenciales del usuario, invoca LSASS y carga el perfil de usuario tras la autenticación. Es el punto de entrada visible.

**LSASS** (`lsass.exe` — Local Security Authority Subsystem Service) es el corazón de la autenticación. Valida credenciales, genera tokens de acceso, aplica políticas de seguridad y, crucialmente, **mantiene en memoria las credenciales de las sesiones activas** para facilitar SSO y autenticación en red. Es el objetivo más valioso para credential dumping.

**SAM** (Security Accounts Manager) es la base de datos local de cuentas de usuario. Almacena los hashes NTLM de las cuentas locales del sistema. Está cifrada con una clave derivada del registro (SYSKEY) y no es directamente legible en caliente.

**Active Directory / NTDS.dit** reemplaza a SAM en entornos de dominio. `NTDS.dit` es la base de datos del Domain Controller que contiene hashes NTLM (y Kerberos) de todos los usuarios del dominio.

### Flujo de autenticación local

```
Usuario introduce credenciales
        ↓
WinLogon captura y envía a LSASS
        ↓
LSASS llama al paquete de autenticación (MSV1_0 para NTLM)
        ↓
MSV1_0 lee el hash de SAM y compara con el hash de la contraseña introducida
        ↓
Si coincide → genera Access Token → sesión iniciada
```

### Flujo de autenticación en dominio

```
Usuario introduce credenciales
        ↓
WinLogon → LSASS → Kerberos SSP
        ↓
Solicita TGT al KDC (Key Distribution Center en el DC)
        ↓
KDC valida con NTDS.dit → emite TGT cifrado con krbtgt
        ↓
Cliente usa TGT para solicitar TGS por cada servicio
        ↓
Servicio valida TGS → acceso concedido
```

### Paquetes de autenticación (SSP/AP)

Windows usa múltiples Security Support Providers (SSP) que LSASS carga en memoria. Cada uno gestiona un protocolo de autenticación diferente y puede almacenar credenciales en distintos formatos:

| SSP        | Protocolo         | Credenciales almacenadas                                              |
| ---------- | ----------------- | --------------------------------------------------------------------- |
| `msv1_0`   | NTLM / LM         | Hashes NT, LM                                                         |
| `kerberos` | Kerberos          | Tickets, claves de sesión                                             |
| `wdigest`  | HTTP Digest       | **Contraseña en claro** (desactivado por defecto desde Win8.1/2012R2) |
| `tspkg`    | Terminal Services | Credenciales de sesión RDP                                            |
| `lsasrv`   | LSAS              | Credenciales de red cacheadas                                         |
| `livessp`  | Microsoft Account | Credenciales de cuenta Live                                           |

> WDigest almacenaba contraseñas en texto claro en LSASS. Aunque está desactivado por defecto en sistemas modernos, puede reactivarse editando el registro. En sistemas legacy (Windows 7, 2008R2 sin parches) WDigest puede estar activo y Mimikatz lo aprovecha directamente.

### NTLM — el protocolo legado

NTLM es el protocolo de autenticación challenge-response que sigue siendo omnipresente en redes Windows por compatibilidad retroactiva. El hash que se almacena y se transmite es el **NT hash** (MD4 del plaintext en UTF-16LE). No usa salt, lo que lo hace crackeble offline y reutilizable en ataques Pass-the-Hash.

| Versión | Vulnerabilidad principal                                                                                  |
| ------- | --------------------------------------------------------------------------------------------------------- |
| LM      | Divide la contraseña en dos bloques de 7 chars, trivialmente crackeable; desactivado en sistemas modernos |
| NTLMv1  | Challenge fijo, vulnerable a relay y cracking                                                             |
| NTLMv2  | Challenge variable (incluye timestamp), más robusto pero aún vulnerable a relay                           |

> Kerberos es el protocolo preferido en dominios modernos, pero NTLM sigue usándose cuando se accede por IP en lugar de nombre, cuando el DC no está disponible, o cuando el servicio no soporta Kerberos.
