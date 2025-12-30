# Enumeracion de usuario

### KERBRUTE

### Identificación de los usuarios

Si nuestro cliente no nos proporciona un usuario con el que empezar a probar (lo que suele ser el caso), tendremos que encontrar una manera de establecer un punto de apoyo en el dominio, ya sea obteniendo credenciales de texto sin cifrar o un hash de contraseña NTLM para un usuario, un shell SYSTEM en un host unido a un dominio o un shell en el contexto de una cuenta de usuario de dominio. Obtener un usuario válido con credenciales es fundamental en las primeras etapas de una prueba de penetración interna. Este acceso (incluso en el nivel más bajo) abre muchas oportunidades para realizar enumeraciones e incluso ataques. A continuación se muestra una forma de comenzar a recopilar una lista de usuarios válidos en un dominio para usarla más adelante en nuestra evaluación.

#### Kerbrute: enumeración interna de nombres de usuario de AD

[Kerbrute](https://github.com/ropnop/kerbrute) puede ser una opción más sigilosa para la enumeración de cuentas de dominio. Aprovecha el hecho de que los errores de autenticación previa de Kerberos a menudo no desencadenarán registros ni alertas. Usaremos Kerbrute junto con las listas de usuarios de [Insidetrust](https://github.com/insidetrust/statistically-likely-usernames). Este repositorio contiene muchas listas de usuarios diferentes que pueden ser extremadamente útiles cuando se intenta enumerar usuarios cuando se comienza desde una perspectiva no autenticada. Podemos apuntar a Kerbrute a la CD que encontramos antes y alimentarla con una lista de palabras. La herramienta es rápida y se nos proporcionarán resultados que nos permitirán saber si las cuentas encontradas son válidas o no, lo cual es un excelente punto de partida para lanzar ataques como la pulverización de contraseñas, que cubriremos en profundidad más adelante en este módulo.

{% stepper %}
{% step %}
### 1) Obtener Kerbrute (descarga o compilar)

Puedes descargar los binarios precompilados desde:\
https://github.com/ropnop/kerbrute/releases/latest

O clonar el repositorio y compilarlo (buena práctica para entornos de cliente).

Clonar el repositorio:

```shell
sudo git clone https://github.com/ropnop/kerbrute.git
```

Salida de ejemplo:

```
Cloning into 'kerbrute'...
remote: Enumerating objects: 845, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 845 (delta 18), reused 28 (delta 10), pack-reused 798
Receiving objects: 100% (845/845), 419.70 KiB | 2.72 MiB/s, done.
Resolving deltas: 100% (371/371), done.
```

Puedes ver las opciones de compilación con:

```shell
make help
```

Salida de ejemplo:

```
help:            Show this help.
windows:  Make Windows x86 and x64 Binaries
linux:  Make Linux x86 and x64 Binaries
mac:  Make Darwin (Mac) x86 and x64 Binaries
clean:  Delete any binaries
all:  Make Windows, Linux and Mac x86/x64 Binaries
```
{% endstep %}

{% step %}
### 2) Compilar (opcional)

Para compilar para múltiples plataformas y arquitecturas:

```shell
sudo make all
```

Salida de ejemplo (recortada):

```
go: downloading github.com/spf13/cobra v1.1.1
...
cd /tmp/kerbrute
rm -f kerbrute kerbrute.exe ...
Done.
Building for windows amd64..
<SNIP>
```

Los binarios compilados quedarán en el directorio `dist/`.
{% endstep %}

{% step %}
### 3) Verificar binarios compilados

Listar los binarios en `dist/`:

```shell
ls dist/
```

Salida de ejemplo:

```
kerbrute_darwin_amd64  kerbrute_linux_386  kerbrute_linux_amd64  kerbrute_windows_386.exe  kerbrute_windows_amd64.exe
```
{% endstep %}

{% step %}
### 4) Probar el binario

Ejecutar el binario para comprobar que funciona (ejemplo con la versión Linux x64):

```shell
./kerbrute_linux_amd64
```

Salida de ejemplo (recortada):

```
    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.
It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.
Warning: failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts

Usage:
  kerbrute [command]
  
  <SNIP>
```
{% endstep %}

{% step %}
### 5) Agregar Kerbrute al PATH (opcional)

Comprobar el PATH:

```shell
echo $PATH
```

Mover el binario a una ruta en PATH, por ejemplo:

```shell
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```
{% endstep %}

{% step %}
### 6) Enumeración de usuarios con Kerbrute

Ejecutar la enumeración de usuarios apuntando al DC y usando una lista de usuarios (por ejemplo `jsmith.txt`), y guardar la salida en `valid_ad_users`:

```shell
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

Salida de ejemplo (recortada):

```
2021/11/17 23:01:46 >  Using KDC(s):
2021/11/17 23:01:46 >   172.16.5.5:88
2021/11/17 23:01:46 >  [+] VALID USERNAME:       jjones@INLANEFREIGHT.LOCAL
2021/11/17 23:01:46 >  [+] VALID USERNAME:       sbrown@INLANEFREIGHT.LOCAL
2021/11/17 23:01:46 >  [+] VALID USERNAME:       tjohnson@INLANEFREIGHT.LOCAL
2021/11/17 23:01:50 >  [+] VALID USERNAME:       evalentin@INLANEFREIGHT.LOCAL
<SNIP>
2021/11/17 23:01:51 >  [+] VALID USERNAME:       sgage@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       jshay@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       jhermann@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       whouse@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       emercer@INLANEFREIGHT.LOCAL
2021/11/17 23:01:52 >  [+] VALID USERNAME:       wshepherd@INLANEFREIGHT.LOCAL
2021/11/17 23:01:56 >  Done! Tested 48705 usernames (56 valid) in 9.940 seconds
```

En el ejemplo validamos 56 usuarios en INLANEFREIGHT.LOCAL en pocos segundos. Estos resultados se pueden usar para crear una lista de usuarios válidos para ataques posteriores, como la pulverización de contraseñas dirigida.
{% endstep %}
{% endstepper %}
