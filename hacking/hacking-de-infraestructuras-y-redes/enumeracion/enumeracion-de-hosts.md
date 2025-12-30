# Enumeración de hosts

El primer paso al enumerar una red es mapear los dispositivos de la red para saber como esta estructurada. Para ello se usan herramienta de escaneo como nmap, masscan, unicornscan entre otras

### Enumeración de hosts con nmap&#x20;

```
pyrhos@debian-laptop:~$ sudo nmap -sn 192.168.1.1/24
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-21 10:34 CET
Nmap scan report for router.asus.com (192.168.1.1)
Host is up (0.0060s latency).
MAC Address: 2C:4D:54:1A:E8:10 (ASUSTek Computer)
Nmap scan report for DESKTOP-P6414SD (192.168.1.230)
Host is up (0.30s latency).
MAC Address: 10:6F:D9:56:CA:6D (Cloud Network Technology Singapore PTE.)
Nmap scan report for OPPO-A16s (192.168.1.238)
Host is up (0.17s latency).
MAC Address: BE:79:7B:15:C2:8F (Unknown)
Nmap scan report for 192.168.1.250
Host is up (0.14s latency).
MAC Address: A2:96:BD:05:DB:A5 (Unknown)
Nmap scan report for 192.168.1.253
Host is up (0.14s latency).
MAC Address: 26:C7:91:EA:78:35 (Unknown)
Nmap scan report for debian-laptop (192.168.1.233)
Host is up.
Nmap done: 256 IP addresses (6 hosts up) scanned in 4.93 seconds
```

También se pueden usar otros parámetros como -sn para hacer otro tipos de escaneo, pero esta fase debe de ser lo mas sigilosa posible

* -sn: hace un escaneo arp
* /24: define el rango de la red en el que nmap va a buscar dispositivos.

