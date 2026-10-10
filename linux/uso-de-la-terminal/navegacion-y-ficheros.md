---
icon: terminal
---

# Navegación y ficheros

### Rutas: absolutas vs relativas

| Notación | Significado                      |
| -------- | -------------------------------- |
| `/`      | Raíz del sistema de ficheros     |
| `~`      | Home del usuario actual          |
| `.`      | Directorio actual                |
| `..`     | Directorio padre                 |
| `-`      | Directorio anterior (con `cd -`) |

```bash
cd /etc/ssh          # ruta absoluta, siempre desde la raíz
cd ../configs          # ruta relativa, desde donde estés
cd ~                    # ir al home
cd -                     # volver al directorio anterior
```

### pwd y cd

```bash
pwd              # muestra el directorio actual (print working directory)
cd                 # sin argumentos, va al home
cd /var/log
cd ..
cd ../..            # subir dos niveles
```

### ls y sus flags útiles

```bash
ls              # listado simple
ls -l           # formato largo: permisos, dueño, tamaño, fecha
ls -la          # incluye ficheros ocultos (empiezan por .)
ls -lh          # tamaños legibles (K, M, G)
ls -lt          # ordenado por fecha de modificación
ls -lS          # ordenado por tamaño
ls -R           # recursivo
ls -d */        # solo directorios
```

#### Lectura de `ls -l`

```
-rwxr-xr-- 1 pyrhos pyrhos 4096 oct 10 12:30 script.sh
```

| Campo           | Significado                                              |
| --------------- | -------------------------------------------------------- |
| `-`             | Tipo (`-` fichero, `d` directorio, `l` enlace simbólico) |
| `rwxr-xr--`     | Permisos: propietario / grupo / otros                    |
| `1`             | Número de enlaces duros                                  |
| `pyrhos pyrhos` | Propietario / grupo                                      |
| `4096`          | Tamaño en bytes                                          |
| `oct 10 12:30`  | Fecha de última modificación                             |

### Crear

```bash
mkdir carpeta              # crear directorio
mkdir -p a/b/c               # crear estructura anidada de golpe
touch fichero.txt            # crear fichero vacío (o actualizar su fecha si existe)
```

### Copiar, mover y borrar

```bash
cp origen destino                   # copiar fichero
cp -r carpeta_origen destino/       # copiar directorio recursivamente
cp -v origen destino                # verbose, muestra qué copia
cp -p origen destino                # preserva permisos y timestamps

mv origen destino                   # mover o renombrar
mv fichero.txt nuevo_nombre.txt     # renombrar es un mv dentro del mismo dir

rm fichero.txt
rm -r carpeta/            # recursivo, necesario para directorios
rm -rf carpeta/           # recursivo + forzado, sin preguntar
rmdir carpeta_vacia/      # solo borra si está vacía
```

&#x20;`rm -rf` no tiene papelera ni confirmación. Comprueba siempre la ruta con `pwd` antes de ejecutar algo destructivo, y nunca lo combines con variables sin comillas (`rm -rf $VAR/*` con `$VAR` vacío borra el directorio actual entero).

### Enlaces simbólicos vs duros

```bash
ln -s /ruta/origen /ruta/enlace     # enlace simbólico (symlink)
ln /ruta/origen /ruta/enlace        # enlace duro (hard link)
```

|                            | Enlace simbólico | Enlace duro       |
| -------------------------- | ---------------- | ----------------- |
| Apunta a                   | Una ruta (texto) | El mismo inodo    |
| Cruza sistemas de ficheros | Sí               | No                |
| Enlaza directorios         | Sí               | No (normalmente)  |
| Si se borra el original    | Queda roto       | Sigue funcionando |

### Ver tipo y tamaño

```bash
file fichero           # identifica el tipo de fichero
du -sh carpeta/          # tamaño total de una carpeta
du -h --max-depth=1      # tamaño de cada subcarpeta, un nivel
df -h                     # espacio en disco por partición
stat fichero               # metadatos completos: permisos, inodo, timestamps
```
