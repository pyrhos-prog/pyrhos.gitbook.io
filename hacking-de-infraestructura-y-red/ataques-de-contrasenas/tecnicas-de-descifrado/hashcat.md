# Hashcat

Hashcat es el cracker de contraseñas más rápido disponible gracias a su aceleración por **GPU mediante OpenCL/CUDA**. Su diseño está optimizado para cracking en volumen: soporta más de 300 tipos de hash y permite combinar modos de ataque con reglas, máscaras y wordlists de forma extremadamente flexible.

### Instalación

```bash
sudo apt install hashcat        # Kali/Debian
hashcat --version
hashcat --example-hashes        # ver ejemplos de hash por tipo
```

### Sintaxis general

```bash
hashcat -m <hash_type> -a <attack_mode> <hashfile> [wordlist/mask]
```

### Tipos de hash (`-m`) más usados

| `-m`    | Algoritmo                  |
| ------- | -------------------------- |
| `0`     | MD5                        |
| `100`   | SHA-1                      |
| `1000`  | NTLM                       |
| `5600`  | Net-NTLMv2                 |
| `5500`  | Net-NTLMv1                 |
| `1800`  | SHA-512crypt (Linux `$6$`) |
| `3200`  | bcrypt                     |
| `13400` | KeePass                    |
| `22000` | WPA-PBKDF2-PMKID+EAPOL     |
| `18200` | AS-REP (Kerberos 5)        |
| `13100` | TGS-REP (Kerberoasting)    |

### Modos de ataque (`-a`)

| `-a` | Modo                 | Descripción                                   |
| ---- | -------------------- | --------------------------------------------- |
| `0`  | Dictionary           | Wordlist sin modificar                        |
| `1`  | Combination          | Combina dos wordlists                         |
| `3`  | Brute-force/Mask     | Prueba todas las combinaciones de una máscara |
| `6`  | Hybrid wordlist+mask | Wordlist con máscara al final                 |
| `7`  | Hybrid mask+wordlist | Máscara al principio + wordlist               |

### Ejemplos prácticos

```bash
# Dictionary attack — NTLM con rockyou
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt

# Dictionary + reglas
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Mask attack — 8 chars, mayúscula + 6 minúsculas + dígito
hashcat -m 1000 -a 3 ntlm.txt '?u?l?l?l?l?l?l?d'

# Hybrid — wordlist + 3 dígitos al final
hashcat -m 1000 -a 6 ntlm.txt rockyou.txt '?d?d?d'

# Kerberoasting TGS
hashcat -m 13100 -a 0 tgs.txt rockyou.txt

# AS-REP Roasting
hashcat -m 18200 -a 0 asrep.txt rockyou.txt
```

### Máscaras — caracteres de posición

| Símbolo | Conjunto                          |
| ------- | --------------------------------- |
| `?l`    | Minúsculas a-z                    |
| `?u`    | Mayúsculas A-Z                    |
| `?d`    | Dígitos 0-9                       |
| `?s`    | Símbolos especiales               |
| `?a`    | Todos los anteriores (`?l?u?d?s`) |
| `?b`    | 0x00-0xff                         |

Se pueden definir conjuntos personalizados con `-1`, `-2`, `-3`, `-4`:

```bash
hashcat -m 1000 -a 3 -1 '?l?d' ntlm.txt '?1?1?1?1?1?1?1?1'
```

### Reglas

Las reglas son ficheros de texto donde cada línea define una transformación. Hashcat incluye un amplio catálogo en `/usr/share/hashcat/rules/`:

* `best64.rule` — 64 reglas de alta efectividad
* `rockyou-30000.rule` — 30.000 reglas derivadas del dataset de RockYou
* `d3ad0ne.rule` — reglas orientadas a contraseñas corporativas

```bash
hashcat -m 1000 -a 0 hashes.txt rockyou.txt -r best64.rule --force
```

### Opciones útiles

```bash
--show                  # mostrar contraseñas ya crackeadas en el potfile
--username              # si el fichero tiene formato usuario:hash
--force                 # ignorar warnings de driver (útil en VM)
--status                # mostrar progreso en tiempo real
-O                      # optimized kernels (más rápido, límite de longitud ~32)
-w 3                    # workload profile (1=bajo, 4=máximo)
--session nombre        # guardar sesión para reanudar
--restore               # reanudar sesión guardada
```

> En máquinas virtuales hashcat puede fallar sin `--force` por problemas de driver OpenCL. En CTF y labs añadir `--force` es habitual, pero en producción conviene ejecutarlo en metal desnudo.

> El potfile (`~/.hashcat/hashcat.potfile`) almacena todos los hashes crackeados. Si un hash ya está en el potfile, hashcat no lo vuelve a procesar; eliminar el potfile o usar `--potfile-disable` para forzar re-cracking.
