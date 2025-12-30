# Puerto 21 (FTP)

**Se puede enumerar el puerto 21 usando nmap con los scripts para FTP:**

```
ftp-anon.nse
- Sirve para comprobar si el inicio de sesión anonimo esta habilitado
ftp-bounce.nse
ftp-brute.nse
- Hace un bruteforce a el FTP víctima
ftp-libopie.nse
ftp-proftpd-backdoor.nse
ftp-syst.nse
ftp-vsftpd-backdoor.nse
ftp-vuln-cve2010-4221.nse
- Comprueba si la version de FTP es vulnerable al cve2010-4221
tftp-enum.nse
tftp-version.nse
- Comprueba la versión de FTP 
```

También se puede usar todos los scripts a la vez de ftp de esta manera:

```
sudo nmap --script=*ftp* <ip-victima>
```

{% hint style="info" %}
Usar este método es mas ruidoso por lo que te pueden detectar y bloquear mas fácilmente.
{% endhint %}

