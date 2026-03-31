# Wordlists y Reglas Personalizadas

La calidad de una wordlist y la inteligencia de las reglas aplicadas determinan en gran medida el éxito del cracking. Usar `rockyou.txt` sin reglas es el punto de partida, pero en entornos corporativos reales las contraseñas suelen seguir patrones que requieren listas y reglas construidas a medida.

### Wordlists de referencia

| Wordlist              | Tamaño         | Uso                                                      |
| --------------------- | -------------- | -------------------------------------------------------- |
| `rockyou.txt`         | \~14M entradas | Punto de partida universal                               |
| `SecLists/Passwords/` | Varias         | Colección categorizada (GitHub: danielmiessler/SecLists) |
| `kaonashi.txt`        | \~450M         | Contraseñas extraídas de múltiples brechas               |
| `weakpass`            | Varios GB      | Colección masiva para cracking serio                     |
| `hashkiller-dict.txt` | \~100M         | Orientada a hashes NTLM/MD5                              |

En Kali, `rockyou.txt` está en `/usr/share/wordlists/rockyou.txt.gz` — descomprimir antes de usar:

```bash
gunzip /usr/share/wordlists/rockyou.txt.gz
```

### Generación de wordlists personalizadas

#### CeWL — spider de sitio web

`CeWL` rastrea un sitio web y extrae palabras únicas para construir una wordlist basada en el vocabulario de la organización objetivo:

```bash
cewl https://empresa.com -d 3 -m 6 -w empresa_wordlist.txt
# -d profundidad de crawl, -m longitud mínima de palabra
```

#### Crunch — generación por patrón

`crunch` genera todas las combinaciones posibles dado un patrón o rango de caracteres:

```bash
crunch 8 8 abcdefghijklmnopqrstuvwxyz0123456789 -o salida.txt
# mínimo 8, máximo 8 chars del charset dado

crunch 6 8 -t Empresa@@@@ -o empresa.txt
# @ = minúscula, , = mayúscula, % = dígito, ^ = símbolo
```

#### CUPP — perfiles de persona

`CUPP` (Common User Passwords Profiler) genera wordlists basadas en información personal del objetivo (nombre, fecha de nacimiento, mascota, pareja):

```bash
pip install cupp
cupp -i        # modo interactivo
```

> &#x20;CUPP solo es válido en engagements donde se ha recabado OSINT previo sobre el target específico.

#### Hashcat `--stdout` — combinar wordlist + reglas para generar nueva lista

```bash
hashcat -a 0 --stdout rockyou.txt -r best64.rule > expanded_wordlist.txt
```

### Reglas en Hashcat

Una regla es una línea que define transformaciones sobre cada candidato. Se pueden encadenar en la misma línea (se aplican secuencialmente):

| Función | Descripción                     |
| ------- | ------------------------------- |
| `l`     | Todo a minúsculas               |
| `u`     | Todo a mayúsculas               |
| `c`     | Capitalizar primera letra       |
| `r`     | Invertir la cadena              |
| `d`     | Duplicar (`pass` → `passpass`)  |
| `$X`    | Añadir carácter X al final      |
| `^X`    | Añadir carácter X al principio  |
| `sXY`   | Sustituir X por Y               |
| `iNX`   | Insertar X en posición N        |
| `dN`    | Eliminar carácter en posición N |
| `'N`    | Truncar a N caracteres          |
| `[`     | Eliminar primer carácter        |
| `]`     | Eliminar último carácter        |

Ejemplo de regla personalizada para patrones corporativos comunes:

```
c$1$!           # Capitalizar + "1!" al final → Password1!
c$2$0$2$4       # → Password2024
sa@se3si!       # Sustituir a→@ e→3 i→! → p@ssw0rd!
```

```bash
# Crear fichero de regla
echo 'c$1$!' > corp_rules.rule
echo 'c$2$0$2$4' >> corp_rules.rule
echo 'sa@se3' >> corp_rules.rule

hashcat -m 1000 -a 0 hashes.txt rockyou.txt -r corp_rules.rule
```

### Reglas en John the Ripper

En JtR la sintaxis es diferente pero el concepto es el mismo. Se definen en `john.conf`:

```
[List.Rules:Corp]
c Az"[0-9!@#]"      # capitalizar + carácter alfanumérico/símbolo al final
Az"[0-9][0-9]"      # dos dígitos al final
```

```bash
john hashes.txt --wordlist=rockyou.txt --rules=Corp
```

> Para entornos AD, combinar `CeWL` sobre la intranet con reglas que añaden el año actual y símbolos (`$2024`, `$2024!`) captura una fracción significativa de contraseñas de políticas corporativas típicas.
