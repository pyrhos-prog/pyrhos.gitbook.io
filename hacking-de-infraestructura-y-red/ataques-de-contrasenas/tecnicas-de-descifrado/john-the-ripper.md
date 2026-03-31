# John the Ripper

John the Ripper (JtR) es una de las herramientas de cracking más veteranas y versátiles. Su punto fuerte es la **detección automática de formato** y el soporte nativo de cientos de tipos de hash sin necesidad de especificar el algoritmo manualmente. También incluye utilidades `*2john` que convierten archivos protegidos (ZIP, PDF, KeePass, SSH keys…) a formato hasheable.

### Instalación

La versión recomendada es **Jumbo**, que incluye todos los módulos adicionales:

```bash
sudo apt install john          # Debian/Kali — versión Jumbo
john --list=formats            # ver todos los formatos soportados
```

### Uso básico

```bash
john hash.txt                              # detección automática de formato
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
john hash.txt --format=NT                 # forzar formato NTLM
john hash.txt --rules                     # aplicar reglas por defecto
john --show hash.txt                      # mostrar contraseñas ya crackeadas
```

Los resultados se guardan en `~/.john/john.pot`. Si se vuelve a ejecutar JtR sobre el mismo hash, lo marca como ya crackeado y no lo reprocesa; para forzar: `--no-pot`.

### Formatos frecuentes

| Tipo                 | Flag `--format` |
| -------------------- | --------------- |
| NTLM (Windows)       | `NT`            |
| Net-NTLMv1           | `netntlm`       |
| Net-NTLMv2           | `netntlmv2`     |
| Linux shadow SHA-512 | `sha512crypt`   |
| bcrypt               | `bcrypt`        |
| MD5                  | `raw-md5`       |
| SHA-1                | `raw-sha1`      |
| KeePass              | `keepass`       |
| ZIP (pkzip)          | `pkzip`         |

### Utilidades \*2john

John incluye scripts para extraer el hash de archivos protegidos antes de crackearlo:

```bash
zip2john archivo.zip > zip.hash
pdf2john documento.pdf > pdf.hash
ssh2john id_rsa > ssh.hash
keepass2john base.kdbx > keepass.hash
office2john documento.docx > office.hash

john zip.hash --wordlist=rockyou.txt
```

### Reglas personalizadas

Las reglas se definen en `/etc/john/john.conf` (o `~/.john/john.conf`) bajo una sección `[List.Rules:NombreRegla]`. La sintaxis mezcla modificadores de posición y transformaciones:

```
[List.Rules:MyRules]
Az"[0-9]"           # añadir dígito al final
A0"[0-9]"           # añadir dígito al principio
c                   # capitalizar primera letra
l                   # todo a minúsculas
u                   # todo a mayúsculas
sa@                 # sustituir 'a' por '@'
```

```bash
john hash.txt --wordlist=rockyou.txt --rules=MyRules
```

> La regla `--rules=KoreLogic` incluida en Jumbo es una de las más completas para entornos corporativos; cubre patrones como `Password1!`, `W3lcome!`, etc.

### Modos de ataque

JtR tiene tres modos principales: **single crack** (usa el nombre de usuario y variantes para generar candidatos), **wordlist** (diccionario con o sin reglas) y **incremental** (brute-force por espacio de caracteres). El modo incremental raramente es práctico sin GPU; para eso es mejor Hashcat con máscaras.

> JtR es CPU-bound. Para hashes rápidos como NTLM o MD5 en volumen, Hashcat con GPU es órdenes de magnitud más rápido.
