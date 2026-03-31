# Importación de Módulos Externos

Metasploit permite extender su base de datos de exploits mediante la importación de módulos externos o el desarrollo de módulos propios en Ruby. Esta capacidad es vital cuando un exploit reciente se publica en bases de datos como ExploitDB pero aún no ha sido integrado en la rama principal de `msfconsole`.

### Importación de Módulos Externos

Cuando un exploit en **ExploitDB** tiene el tag `Metasploit Framework (MSF)`, significa que el archivo `.rb` está diseñado para funcionar directamente en el framework.

#### Búsqueda con Searchsploit

Podemos filtrar por el nombre del servicio y la extensión `.rb` para localizar módulos compatibles sin salir de la terminal.

```bash
searchsploit nagios3 -t --exclude=".py"
```

#### Instalación y Rutas

Para que Metasploit reconozca un nuevo módulo, este debe guardarse siguiendo la estructura jerárquica del framework (categoría/plataforma/servicio).

* Ruta del sistema: `/usr/share/metasploit-framework/modules/`
* Ruta de usuario: `~/.msf4/modules/` (Recomendado para evitar sobrescrituras en actualizaciones).

> Si la carpeta en `~/.msf4/modules/` no existe, se debe crear manualmente manteniendo la estructura de directorios (ej: `exploits/linux/http/`).

> El nombre del archivo debe seguir la convención snake\_case (ej: `nagios3_command_injection.rb`), usando solo caracteres alfanuméricos y guiones bajos.

### Carga de Módulos en Runtime

Una vez copiado el archivo al directorio correspondiente, existen tres formas de cargarlo en `msfconsole`:

1. Desde el inicio: Lanzar la consola especificando una ruta adicional con el flag `-m`.
2. Durante la sesión: Usar el comando `loadpath` seguido de la ruta del directorio de módulos.
3. Refresco total: Ejecutar `reload_all` dentro de la consola para re-escanear todos los directorios.

Bash

```
# Opción 1: Al iniciar
msfconsole -m ~/.msf4/modules/

# Opción 2: Dentro de msfconsole
msf6 > loadpath /usr/share/metasploit-framework/modules/
msf6 > reload_all
```

### Portado de Scripts a Módulos MSF

Para convertir un exploit (Python, PHP, etc.) en un módulo de Metasploit, se utiliza Ruby y se hereda de la clase `Msf::Exploit::Remote`. Es común usar módulos existentes como _boilerplate_ para ahorrar tiempo.

#### Mixins Comunes

Los _mixins_ añaden funcionalidades específicas al módulo (como gestión de conexiones HTTP o payloads).

| **Mixin**                          | **Descripción**                                               |
| ---------------------------------- | ------------------------------------------------------------- |
| `Msf::Exploit::Remote::HttpClient` | Proporciona métodos para actuar como cliente HTTP             |
| `Msf::Exploit::PhpEXE`             | Genera un payload PHP de primera etapa                        |
| `Msf::Exploit::FileDropper`        | Maneja la transferencia y limpieza de archivos en el target   |
| `Msf::Auxiliary::Report`           | Permite reportar datos directamente a la base de datos de MSF |

#### Estructura del Módulo (MetasploitModule)

El código se divide principalmente en tres bloques de configuración:

1. Metadatos en `initialize`: Se define el nombre, descripción, autor, CVE y referencias.
2. Definición de Targets: Se especifican las versiones de software o sistemas operativos vulnerables.
3. Registro de Opciones: Se definen las variables que el usuario configurará (RHOSTS, TARGETURI, etc.) mediante `register_options`.

> El uso de `OptPath` permite que el módulo cargue diccionarios externos, útil en ataques de bypass de mitigación de fuerza bruta.

#### Ejemplo de Bloque de Opciones

Ruby

```
register_options(
  [
    OptString.new('TARGETURI', [true, 'Ruta base de la aplicación', '/']),
    OptString.new('BLUDITUSER', [true, 'Usuario para el bypass']),
    OptPath.new('PASSWORDS', [true, 'Wordlist de contraseñas', File.join(Msf::Config.data_directory, "wordlists", "passwords.txt")])
  ])
```

Para la ejecución real, el método `exploit` (o métodos auxiliares en módulos `auxiliary`) contendrá la lógica de envío de paquetes, payloads y manejo de la respuesta del servidor.
