---
icon: terminal
---

# Gestión de almacenamiento

### Discos y particiones

#### Ver discos y particiones

```bash
lsblk                     # árbol de discos y particiones, forma más clara
lsblk -f                    # igual, + sistema de ficheros y UUID
fdisk -l                      # listado detallado (requiere privilegios)
parted -l                       # alternativa a fdisk, soporta GPT mejor
blkid                             # UUID y tipo de sistema de ficheros de cada partición
```

#### Espacio en disco

```bash
df -h                     # espacio libre/usado por partición montada, legible
df -hT                      # igual, + tipo de sistema de ficheros

du -sh carpeta/                # tamaño total de una carpeta
du -h --max-depth=1 /          # tamaño de cada subcarpeta de raíz, un nivel — útil para encontrar qué ocupa espacio
du -sh * | sort -rh              # ordenar carpetas del directorio actual por tamaño, de mayor a menor
```

{% hint style="info" %}
`df` mide espacio de **particiones montadas**; `du` mide espacio **real ocupado por ficheros/carpetas**. Pueden no coincidir (ej. un fichero borrado pero aún abierto por un proceso sigue ocupando espacio que `df` cuenta pero `du` no ve).
{% endhint %}

#### Montar y desmontar

```bash
mount /dev/sdb1 /mnt/usb         # montar una partición en un punto existente
umount /mnt/usb                    # desmontar
umount -l /mnt/usb                   # desmontaje "lazy", si está ocupado y no se puede al instante

mount | grep /dev/sdb1                 # comprobar si ya está montado y dónde
```

#### Montaje persistente: /etc/fstab

```bash
cat /etc/fstab
```

```
UUID=xxxx-xxxx  /mnt/datos  ext4  defaults  0  2
```

{% hint style="warning" %}
Un error de sintaxis en `/etc/fstab` puede impedir que el sistema arranque correctamente. Probar primero con `mount -a` (monta todo lo definido en fstab) antes de reiniciar, para detectar errores sin tener que recuperar el sistema después.&#x20;
{% endhint %}

#### Crear particiones y sistemas de ficheros (operación destructiva)

```bash
fdisk /dev/sdb              # editor interactivo de particiones (n=nueva, d=borrar, w=escribir cambios)
mkfs.ext4 /dev/sdb1            # formatear una partición como ext4
mkfs.ntfs /dev/sdb1              # formatear como NTFS (requiere ntfs-3g)
```

{% hint style="info" %}
&#x20;`fdisk` y `mkfs` son operaciones destructivas sobre la partición indicada. Comprobar siempre con `lsblk` qué dispositivo es cuál antes de ejecutar nada — confundir `/dev/sda` (disco del sistema) con `/dev/sdb` (un USB) es un error clásico y catastrófico.&#x20;
{% endhint %}

#### Discos USB y dispositivos extraíbles

```bash
lsblk                       # tras conectar el USB, identifica el nuevo dispositivo (ej. /dev/sdc1)
udisksctl mount -b /dev/sdc1   # montar sin privilegios root, en entornos de escritorio modernos
```
