---
icon: building-magnifying-glass
---

# Técnicas de descifrado

Los sistemas operativos y aplicaciones nunca deberían almacenar contraseñas en texto claro. En su lugar utilizan funciones hash criptográficas que transforman el plaintext en una cadena de longitud fija que no puede revertirse directamente. El objetivo del cracking es recrear ese proceso hacia atrás, probando candidatos hasta encontrar uno cuyo hash coincida.

### Almacenamiento de contraseñas

| Sistema             | Mecanismo                                     | Ubicación habitual                     |
| ------------------- | --------------------------------------------- | -------------------------------------- |
| Linux               | SHA-512 con salt (shadow)                     | `/etc/shadow`                          |
| Windows local       | NTLM hash                                     | SAM (`C:\Windows\System32\config\SAM`) |
| Active Directory    | NTLM / Kerberos                               | `NTDS.dit` en el DC                    |
| Aplicaciones web    | bcrypt, Argon2, PBKDF2 (si bien configuradas) | Base de datos                          |
| Aplicaciones legacy | MD5, SHA-1 sin salt                           | Base de datos                          |

### Tipos de ataque

**Dictionary attack** — se prueba cada entrada de una wordlist tal cual. Efectivo contra contraseñas comunes o basadas en palabras reales.

**Brute-force** — se prueban todas las combinaciones posibles de un espacio de caracteres definido. Garantizado pero inviable para contraseñas largas sin GPU potente.

**Rule-based attack** — se aplican reglas de mutación a una wordlist (añadir números al final, capitalizar, sustituir `a`→`@`, etc.). Es el modo más eficiente en la práctica.

**Mask attack** — brute-force guiado por una máscara que describe la estructura de la contraseña (`?u?l?l?l?d?d` = mayúscula + 3 minúsculas + 2 dígitos).

**Hybrid attack** — combina wordlist con máscara. Por ejemplo, `Password` + `?d?d?d` prueba `Password001` … `Password999`.

**Rainbow tables** — tablas precalculadas de hash→plaintext. Ineficaces contra hashes con salt, pero devastadoras contra MD5/SHA-1 sin salt.

### Identificación de hashes

Antes de atacar hay que saber qué tipo de hash se tiene. `hashid` y `hash-identifier` automatizan esto, aunque los patrones visuales también ayudan:

```bash
hashid '$6$xyz$...'          # SHA-512 crypt (Linux shadow)
hashid '5f4dcc3b5aa765d61d8327deb882cf99'   # MD5
```

| Patrón          | Algoritmo probable |
| --------------- | ------------------ |
| 32 hex chars    | MD5 o NTLM         |
| 40 hex chars    | SHA-1              |
| 64 hex chars    | SHA-256            |
| `$1$`           | MD5crypt           |
| `$2y$` / `$2b$` | bcrypt             |
| `$5$`           | SHA-256crypt       |
| `$6$`           | SHA-512crypt       |
| `$y$`           | yescrypt           |

> NTLM y MD5 sin salt son hash rápidos. Una GPU moderna puede probar miles de millones por segundo. Bcrypt, Argon2 y yescrypt están diseñados para ser lentos; el cracking es mucho menos viable.

### Hardware y velocidades de referencia

La GPU acelera el cracking de forma masiva frente a la CPU. Una RTX 3090 alcanza aproximadamente:

* NTLM: \~140.000 MH/s
* MD5: \~90.000 MH/s
* bcrypt (cost 10): \~25 kH/s

> Para cracking en entornos de lab sin GPU potente, priorizar wordlists pequeñas + reglas sobre brute-force puro.
