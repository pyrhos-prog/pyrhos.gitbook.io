# Descifrado de Archivos Protegidos

Durante un engagement es frecuente encontrar archivos protegidos por contraseña: documentos Office, PDFs, ZIPs, bases de datos KeePass, claves SSH cifradas o archivos 7-Zip. El flujo siempre es el mismo: extraer el hash del archivo con una utilidad `*2john`, e identificar el tipo de hash correcto para Hashcat o JtR.

### Flujo general

```
archivo protegido → *2john / herramienta extracción → hash.txt → hashcat / john → plaintext
```

### Búsqueda de Diferentes tipos de archivos en linux

#### Búsqueda de archivos encriptados

```bash
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

#### Búsqueda de claves SSH

```bash
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

### ZIP y archivos comprimidos

```bash
zip2john archivo.zip > zip.hash
john zip.hash --wordlist=rockyou.txt

# En Hashcat, el tipo depende del cifrado del ZIP:
# -m 13600  WinZip AES
# -m 17220  PKZIP (antiguo)
hashcat -m 13600 zip.hash rockyou.txt
```

> Los ZIPs con cifrado clásico (ZipCrypto) son vulnerables a ataques known-plaintext si se conoce el contenido de al menos un archivo sin cifrar del mismo ZIP. La herramienta `bkcrack` explota esto.

### Documentos Office (Word, Excel, PowerPoint)

```bash
office2john documento.docx > office.hash
john office.hash --wordlist=rockyou.txt

# Hashcat
# -m 9600  MS Office 2013
# -m 9500  MS Office 2010
# -m 9400  MS Office 2007
hashcat -m 9600 office.hash rockyou.txt
```

### PDF

```bash
pdf2john documento.pdf > pdf.hash
john pdf.hash --wordlist=rockyou.txt

# Hashcat: -m 10500 (PDF 1.4-1.6), -m 10700 (PDF 1.7 L8)
hashcat -m 10500 pdf.hash rockyou.txt
```

### KeePass

Las bases de datos KeePass (`.kdbx`) son un objetivo de alto valor; a menudo contienen credenciales corporativas:

```bash
keepass2john base.kdbx > keepass.hash
john keepass.hash --wordlist=rockyou.txt

# Hashcat: -m 13400
hashcat -m 13400 keepass.hash rockyou.txt
```

> 💡 KeePass 2.x con AES-KDF (iteraciones altas) es lento de crackear. Si las iteraciones son bajas o se usa ChaCha20-KDF de versiones antiguas, el cracking es más viable.

### Claves SSH cifradas

Una clave privada SSH (`id_rsa`) puede estar protegida con passphrase:

```bash
ssh2john id_rsa > ssh.hash
john ssh.hash --wordlist=rockyou.txt

# Hashcat: -m 22921 (RSA/DSA/EC/OpenSSH)
hashcat -m 22921 ssh.hash rockyou.txt
```

### 7-Zip

```bash
7z2john archivo.7z > 7z.hash
# Limpiar posibles líneas de metadatos si john se queja
john 7z.hash --wordlist=rockyou.txt

# Hashcat: -m 11600
hashcat -m 11600 7z.hash rockyou.txt
```

### Hashes de autenticación de red — resumen rápido

Cuando se capturan hashes de red con Responder o similares, el proceso es:

```bash
# Net-NTLMv2 capturado con Responder
hashcat -m 5600 netntlmv2.txt rockyou.txt -r best64.rule

# Net-NTLMv1
hashcat -m 5500 netntlmv1.txt rockyou.txt
```

### Tabla resumen de modos Hashcat para archivos

| Tipo de archivo    | `-m` Hashcat |
| ------------------ | ------------ |
| ZIP (WinZip AES)   | `13600`      |
| ZIP (PKZIP legacy) | `17220`      |
| 7-Zip              | `11600`      |
| Office 2013        | `9600`       |
| Office 2010        | `9500`       |
| PDF 1.4-1.6        | `10500`      |
| KeePass            | `13400`      |
| SSH private key    | `22921`      |

> Para identificar rápidamente el tipo de hash de un archivo extraído con `*2john`, el formato del hash suele incluir el nombre del tipo entre `$` signs, que Hashcat reconoce con `--example-hashes | grep <pattern>`.
