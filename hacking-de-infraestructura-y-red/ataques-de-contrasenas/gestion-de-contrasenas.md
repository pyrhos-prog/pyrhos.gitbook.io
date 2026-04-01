---
icon: building-magnifying-glass
---

# Gestión de contraseñas

## Políticas de Contraseñas

Las políticas de contraseñas definen los requisitos mínimos que deben cumplir las credenciales en un sistema o dominio. Para un atacante, conocer la política antes de un ataque de spraying o cracking reduce el espacio de búsqueda enormemente; para un defensor, una política bien calibrada eleva significativamente el coste de esos ataques.

### Componentes de una política de contraseñas

| Parámetro             | Descripción                             | Recomendación NIST SP 800-63B                                   |
| --------------------- | --------------------------------------- | --------------------------------------------------------------- |
| **Longitud mínima**   | Mínimo de caracteres                    | ≥ 8 (usuarios), ≥ 15 (admins)                                   |
| **Complejidad**       | Requerir mayús/minús/dígitos/símbolos   | NIST recomienda **no** forzarla — favorece patrones predecibles |
| **Historial**         | No reutilizar las últimas N contraseñas | ≥ 12                                                            |
| **Caducidad**         | Días hasta forzar cambio                | NIST recomienda **no** forzar caducidad periódica               |
| **Lockout threshold** | Intentos fallidos antes de bloqueo      | 5-10 intentos                                                   |
| **Lockout duration**  | Tiempo de bloqueo o reseteo manual      | 15-30 min (automático) o manual                                 |

> El NIST ya no recomienda forzar cambios periódicos de contraseña ni complejidad artificial. Ambas medidas provocan comportamientos predecibles: `Password1!` → `Password2!` → `Password3!`. La longitud es el factor más importante para resistir ataques de fuerza bruta.

### Políticas en Active Directory

#### Fine-Grained Password Policies (FGPP)

En AD se pueden aplicar políticas distintas a grupos o usuarios específicos, más restrictivas para cuentas privilegiadas:

```powershell
# Ver la política de dominio por defecto
Get-ADDefaultDomainPasswordPolicy

# Ver políticas granulares
Get-ADFineGrainedPasswordPolicy -Filter *

# Ver qué política aplica a un usuario concreto
Get-ADUserResultantPasswordPolicy -Identity jsmith
```

Desde CMD:

```cmd
net accounts /domain
```

#### Enumeración de la política (sin credenciales)

```bash
# Con Impacket — sin autenticación si la política lo permite
impacket-rpcdump 192.168.1.10 | grep -i policy

# Con NetExec — con credenciales
netexec smb 192.168.1.10 -u usuario -p Password123 --pass-pol
```

Salida típica de `--pass-pol`:

```
[+] Dumping password policy
Minimum password length: 7
Password history length: 24
Maximum password age: 42 days
Password complexity: Enabled
Lockout threshold: 5
Lockout duration: 30 minutes
Lockout observation window: 30 minutes
```

### Implicaciones para el atacante

Antes de lanzar un ataque de spraying sobre un dominio:

1. Recuperar la política (`--pass-pol`)
2. Identificar el umbral de lockout (p. ej., 5 intentos)
3. Usar **umbral - 1** contraseñas por ronda (p. ej., máximo 4 intentos por usuario)
4. Respetar la ventana de observación entre rondas
5. Priorizar contraseñas que cumplan la política: si requiere complejidad, usar `Password2024!` en lugar de `password`

### Contraseñas corporativas predecibles

Aunque la política fuerce complejidad, los usuarios tienden a usar patrones que la satisfacen con el mínimo esfuerzo. Las más efectivas para spraying en entornos corporativos:

* `NombreEmpresa2024!`
* `Welcome1!` / `Welcome2024`
* `Password1!` / `P@ssw0rd`
* `Changeme1!` / `Change123`
* `Temporada + año + símbolo`: `Verano2024!`, `Enero2024!`
* Nombre del dominio capitalizado + año + símbolo

> En dominios donde el nombre del dominio es `Empresa`, las contraseñas `Empresa2024!` o `Empresa1!` tienen tasa de éxito sorprendentemente alta en spraying, especialmente en cuentas recién creadas o de nuevos empleados.

### Password Auditing — auditar la calidad de contraseñas del dominio

```bash
# Extraer hashes del dominio y auditar con herramientas especializadas
# DPAT (Domain Password Audit Tool)
dpat.py -n ntds.dit -c SYSTEM -o reporte/

# Hashcat con estadísticas
hashcat -m 1000 domain_hashes.txt rockyou.txt --outfile cracked.txt
# Calcular % crackeado
wc -l cracked.txt domain_hashes.txt
```

### Controles complementarios

| Control                  | Efecto sobre ataques de contraseña                            |
| ------------------------ | ------------------------------------------------------------- |
| **MFA**                  | Hace inútil una contraseña comprometida sin el segundo factor |
| **LAPS**                 | Elimina reutilización de admin local entre hosts              |
| **Credential Guard**     | Protege hashes en LSASS frente a dumping                      |
| **Protected Users**      | Fuerza Kerberos, desactiva NTLM, WDigest y delegación         |
| **Banned password list** | Rechaza contraseñas conocidas (Azure AD Password Protection)  |

## Gestores de Contraseñas

Los gestores de contraseñas son la herramienta más efectiva disponible para usuarios individuales frente a los ataques de credential stuffing y reutilización. Generan y almacenan contraseñas únicas y aleatorias por cuenta, eliminando el vector de reutilización. Para un pentester son también objetivos de alto valor: comprometer el gestor de un administrador puede dar acceso a todas las credenciales corporativas de golpe.

### Por qué importan — el problema que resuelven

El 23% de usuarios reutiliza contraseñas en tres o más cuentas. Un gestor elimina esa reutilización porque el usuario solo necesita recordar una contraseña maestra: el gestor genera y recuerda contraseñas de 20+ caracteres aleatorios por cada servicio. Una contraseña como `f7K#mP2$qXw9!nL4@vRt` es imposible de memorizar pero trivial de almacenar en un gestor.

### Tipos de gestores

| Tipo                        | Almacenamiento                | Ejemplos                                    | Riesgo principal                                    |
| --------------------------- | ----------------------------- | ------------------------------------------- | --------------------------------------------------- |
| **Local**                   | Archivo cifrado en disco      | KeePass, KeePassXC                          | Pérdida del archivo o de la master password         |
| **Cloud/SaaS**              | Servidor del proveedor        | Bitwarden, 1Password, Dashlane              | Brecha en el proveedor, phishing de master password |
| **Integrado en OS/browser** | Keychain, Credential Manager  | iCloud Keychain, Windows Credential Manager | Acceso físico o malware con privilegios             |
| **Empresarial**             | On-premise o cloud gestionado | CyberArk, Hashicorp Vault, Delinea          | Configuración incorrecta, acceso admin comprometido |

### KeePass — referencia técnica

KeePass es el gestor open source más extendido en entornos corporativos. Las bases de datos `.kdbx` se cifran con AES-256 (KeePass 2.x) usando la master password y opcionalmente un key file o la cuenta de Windows como factor adicional.

```bash
# Desde perspectiva ofensiva — cracking de .kdbx
keepass2john base.kdbx > keepass.hash
hashcat -m 13400 keepass.hash rockyou.txt

# Con John
john keepass.hash --wordlist=rockyou.txt
```

> El número de iteraciones KDF configurable en KeePass 2.x determina la velocidad de cracking. Una base de datos con iteraciones altas (≥ 100.000) y una master password de 12+ caracteres es prácticamente inmune al cracking con hardware convencional.

#### Extracción de la base de datos en caliente

Si KeePass está abierto en memoria, la master password puede recuperarse con técnicas de memory forensics:

```bash
# KeePassXC-Dump — extraer master password de la memoria del proceso
# (requiere acceso como root o el mismo usuario)
sudo strings /proc/$(pgrep keepassxc)/mem 2>/dev/null | grep -i "master"

# Herramienta dedicada: keepass-dump-masterkey
# Funciona sobre volúmenes de memoria o procesos en vivo
```

### Bitwarden — referencia técnica

Bitwarden es la alternativa open source cloud más popular. El vault se cifra localmente antes de sincronizar al servidor (zero-knowledge por diseño). La clave de cifrado deriva de la master password con PBKDF2-SHA256 (mínimo 600.000 iteraciones desde 2023).

Desde perspectiva ofensiva, el vault de Bitwarden en disco local:

```bash
# Ubicación del vault local en Linux
~/.config/Bitwarden/data.json

# En Windows
%AppData%\Bitwarden\data.json
```

El archivo contiene el vault cifrado; sin la master password no es utilizable. El vector más realista es phishing de la master password o interceptar el unlock en sesión activa con un keylogger.

### Gestores empresariales — objetivos de alto valor

En un pentest interno, los gestores de contraseñas corporativos son objetivos prioritarios:

| Gestor                             | Puerto/Interfaz | Vector                                      |
| ---------------------------------- | --------------- | ------------------------------------------- |
| **CyberArk PAM**                   | HTTPS, API REST | Credenciales de administrador del vault     |
| **HashiCorp Vault**                | TCP 8200        | Token de acceso, políticas mal configuradas |
| **Thycotic/Delinea Secret Server** | HTTPS           | Credenciales de dominio con acceso al vault |

```bash
# HashiCorp Vault — verificar acceso con token
curl -H "X-Vault-Token: TOKEN" http://vault.empresa.local:8200/v1/secret/data/

# Enumerar secrets accesibles
vault list secret/
```

### Mejores prácticas para blue team

* Master password de mínimo 16 caracteres (frase completa recomendada sobre cadena aleatoria)
* Activar MFA en gestores cloud (TOTP o hardware key)
* Usar key file adicional en KeePass para entornos de alta seguridad
* No almacenar el key file en la misma ubicación que el `.kdbx`
* Backups cifrados del vault en ubicación separada
* Auditorías periódicas de contraseñas débiles, reutilizadas o comprometidas (los gestores modernos lo ofrecen nativamente)

> Para entornos corporativos, la combinación de un gestor empresarial (CyberArk, Delinea) para cuentas privilegiadas + Bitwarden o 1Password para cuentas de usuarios estándar + LAPS para admins locales cubre la mayor parte de la superficie de ataque de credenciales.
