# Puerto 22 (SSH)

### Enumeración con nmap

Para enumerar el servicio SSH se pueden usar los siguientes scripts de nmap:

```
ssh2-enum-algos.nse
ssh-auth-methods.nse
ssh-brute.nse
ssh-hostkey.nse
ssh-publickey-acceptance.nse
ssh-run.nse
sshv1.nse
```

O también&#x20;

```
pyrhos@debian-laptop:~$ nmap -p22 --script=*ssh* <ip-víctima>
```
