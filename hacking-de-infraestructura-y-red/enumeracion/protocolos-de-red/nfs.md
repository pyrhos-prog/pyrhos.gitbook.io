---
icon: building-magnifying-glass
---

# NFS

NFS (Network File System) es un sistema de archivos distribuido desarrollado originalmente por Sun Microsystems.

Permite montar directorios remotos como si fueran locales.

Se utiliza principalmente entre sistemas Linux y Unix.

Puertos relevantes:

* 111/tcp y 111/udp → rpcbind
* 2049/tcp y 2049/udp → NFS
* Puertos dinámicos adicionales (mountd, nlockmgr, etc.) en NFSv3
* Solo 2049 en NFSv4

**La enumeración NFS se centra en:**

* Identificar exports disponibles
* Detectar configuraciones inseguras
* Analizar UID/GID
* Buscar archivos sensibles
* Evaluar posibilidad de escalada

### Versiones

* **NFSv2**: UDP, obsoleto, sin seguridad moderna.
* **NFSv3**: Archivos grandes, usa rpcbind (111), autenticación por UID/GID.
* **NFSv4**: Stateful, puerto 2049, soporte Kerberos y ACLs.
* **NFSv4.1**: pNFS y mejoras para entornos distribuidos.

### Seguridad

* Basado en ONC-RPC.
* El servidor confía en los UID/GID enviados por el cliente.
* Riesgo si los mapeos no coinciden.
* Preferible usar Kerberos en entornos no confiables.

### Archivo de Configuración

Ruta principal:

```
/etc/exports
```

Ejemplo:

```
/mnt/nfs 10.129.14.0/24(sync,no_subtree_check)
```

Formato:

```
<directorio> <host/subred>(opciones)
```

## /etc/exports – Opciones Clave

{% tabs %}
{% tab title="Permisos" %}
**rw** → Lectura y escritura\
**ro** → Solo lectura
{% endtab %}

{% tab title="Sincronización" %}
* `sync` → escritura síncrona
* `async` → escritura asíncrona
{% endtab %}

{% tab title="Puertos" %}
* `secure` → solo <1024
* `insecure` → permite >1024
{% endtab %}

{% tab title="Subdirectorios" %}
* `no_subtree_check` → desactiva validación
{% endtab %}

{% tab title="Root" %}
* `root_squash` → UID 0 → usuario anónimo
* `no_root_squash` → mantiene UID 0 real
{% endtab %}

{% tab title="Configuración crítica" %}
* `rw`, `insecure`, `nohide`, `no_root_squash`
* `no_root_squash` permite que un root remoto conserve UID 0 → alto riesgo de escalada.
{% endtab %}
{% endtabs %}

### Footprinting del Servicio

Puertos clave:

* 111
* 2049

Escaneo básico:

```bash
nmap -p111,2049 -sV -sC <IP>
```

Extrae:

* Servicios RPC activos
* Versiones NFS
* Program numbers

### Enumeración RPC

El script rpcinfo lista servicios RPC:

```bash
rpcinfo -p <IP>
```

Permite identificar:

* nfs
* mountd
* nlockmgr
* nfs\_acl

### Enumeración con Scripts NSE

```bash
nmap --script nfs* -p111,2049 -sV <IP>
```

Scripts relevantes:

* nfs-ls
* nfs-showmount
* nfs-statfs

Permite:

* Listar contenido remoto
* Mostrar exports
* Obtener estadísticas del filesystem

### Enumerar Shares Disponibles

```bash
showmount -e <IP>
```

Devuelve:

* Directorios exportados
* Subredes autorizadas

### Montar un NFS Share

Crear directorio local:

```bash
mkdir target-NFS
```

Montar:

```bash
sudo mount -t nfs <IP>:/ ./target-NFS -o nolock
```

Acceder:

```bash
cd target-NFS
```

### Enumeración de Archivos

Listar con nombres:

```bash
ls -l
```

Listar con UID/GID:

```bash
ls -n
```

Información clave:

* UID
* GID
* Permisos
* Archivos sensibles

### Manipulación de UID/GID

Si root\_squash no está habilitado:

Se puede crear un usuario local con mismo UID que el propietario remoto.

Ejemplo:

```bash
sudo useradd -u 1000 attacker
```

Permite heredar permisos sobre archivos exportados.

### Escalada de Privilegios vía NFS

Escenario típico:

* Export con no\_root\_squash
* Permiso rw

Procedimiento conceptual:

* Crear binario con SUID
* Subirlo al share
* Ejecutarlo desde la máquina víctima

Esto permite obtener privilegios elevados.

### Indicadores Críticos Durante Enumeración

* no\_root\_squash habilitado
* rw para toda la subred
* insecure habilitado
* Archivos sensibles expuestos
* Claves SSH privadas en share
* Scripts de backup accesibles

### Diferencia NFSv3 vs NFSv4 en Seguridad

NFSv3:

* Basado en host
* Uso de puertos dinámicos
* Sin autenticación fuerte

NFSv4:

* Soporte Kerberos
* Stateful
* Mejor control ACL
* Menor superficie de puertos

### Desmontar el Share

```bash
sudo umount ./target-NFS
```
