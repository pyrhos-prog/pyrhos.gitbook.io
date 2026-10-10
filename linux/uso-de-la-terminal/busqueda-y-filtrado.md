---
icon: terminal
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Búsqueda y filtrado

### find

```bash
find /ruta -name "*.conf"              # por nombre (sensible a mayúsculas)
find /ruta -iname "*.conf"              # por nombre, insensible a mayúsculas
find /ruta -type f                       # solo ficheros
find /ruta -type d                       # solo directorios
find /ruta -type l                       # solo enlaces simbólicos
```

#### Por permisos, fecha y tamaño

```bash
find / -perm -4000 -type f 2>/dev/null   # binarios con SUID
find / -perm -2000 -type f 2>/dev/null   # binarios con SGID
find . -mtime -7                          # modificados en los últimos 7 días
find . -mtime +30                          # modificados hace más de 30 días
find . -size +100M                          # ficheros de más de 100 MB
find . -size -1k                             # ficheros de menos de 1 KB
find . -newer referencia.txt                   # más recientes que un fichero dado
```

#### Combinando condiciones

```bash
find / -user root -perm -4000 -type f 2>/dev/null   # SUID + propietario root
find . -name "*.log" -type f -mtime +30 -delete        # borra logs antiguos (cuidado)
```

#### Ejecutar una acción sobre cada resultado: `-exec`

```bash
find . -name "*.sh" -exec chmod +x {} \;        # añade +x a todos los .sh encontrados
find . -name "*.tmp" -exec rm {} \;               # borra cada .tmp encontrado
find . -name "*.conf" -exec grep -l "password" {} \;   # busca "password" dentro de cada .conf
```

\{% hint style="info" %\} `{}` es el placeholder del fichero encontrado, `\;` termina el comando por cada resultado (uno a uno). `+` en lugar de `\;` agrupa varios resultados en una sola invocación del comando, más eficiente para operaciones masivas. \{% endhint %\}

### grep

```bash
grep "patrón" fichero.txt
grep -i "patrón" fichero.txt         # insensible a mayúsculas
grep -r "patrón" carpeta/              # recursivo
grep -v "patrón" fichero.txt             # invierte: líneas que NO coinciden
grep -n "patrón" fichero.txt               # muestra número de línea
grep -c "patrón" fichero.txt                 # cuenta coincidencias
grep -l "patrón" *.txt                         # solo nombres de ficheros con coincidencia
grep -E "regex extendida" fichero.txt            # regex extendida (equivalente a egrep)
grep -A3 -B3 "patrón" fichero.txt                  # 3 líneas después (After) y antes (Before)
```

| Flag | Significado                            |
| ---- | -------------------------------------- |
| `-i` | Ignora mayúsculas/minúsculas           |
| `-r` | Recursivo en subdirectorios            |
| `-v` | Invierte: líneas que NO coinciden      |
| `-n` | Muestra número de línea                |
| `-c` | Cuenta coincidencias                   |
| `-l` | Solo nombres de fichero con match      |
| `-E` | Regex extendida                        |
| `-w` | Solo coincidencias de palabra completa |

#### Regex básicas con grep

```bash
grep -E "^[0-9]+" fichero.txt         # líneas que empiezan por un número
grep -E "error|fail|critical" log.txt   # cualquiera de varias palabras
grep -E "192\.168\.[0-9]+\.[0-9]+" log.txt   # patrón de IP
```

### locate, which, whereis

```bash
locate nombre_fichero        # búsqueda rápida usando una base de datos indexada
updatedb                      # actualiza la base de datos de locate (necesita privilegios)

which comando                  # ruta del ejecutable que se usaría
whereis comando                  # ruta del binario, fuentes y páginas de manual
```

\{% hint style="warning" %\} `locate` es muy rápido porque consulta un índice (`updatedb`), no el disco en tiempo real. Si acabas de crear un fichero, puede que `locate` aún no lo encuentre hasta la próxima actualización del índice. `find` siempre busca en tiempo real, más lento pero siempre actualizado. \{% endhint %\}

### awk y sed — lo mínimo útil

#### awk: extraer columnas

```bash
awk '{print $1}' fichero.txt           # primera columna
awk -F: '{print $1}' /etc/passwd         # separador distinto (ej: ":")
awk '{print $1, $3}' fichero.txt           # columnas 1 y 3
awk '/error/' log.txt                        # líneas que contienen "error"
```

#### sed: sustituir texto

```bash
sed 's/antiguo/nuevo/' fichero.txt        # sustituye la 1ª ocurrencia por línea
sed 's/antiguo/nuevo/g' fichero.txt       # sustituye todas las ocurrencias
sed -i 's/antiguo/nuevo/g' fichero.txt    # modifica el fichero in-place
sed -n '/patrón/p' fichero.txt              # imprime solo líneas que coinciden
```

{% hint style="info" %}
`-i` sobrescribe el fichero original sin preguntar. Probar siempre primero sin `-i` para ver el resultado por pantalla.&#x20;
{% endhint %}

