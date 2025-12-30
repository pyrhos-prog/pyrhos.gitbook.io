# Enumeración de puertos y servicios

Después de tener el mapa de la red, hay que enumerar los puertos y los servicios del host para buscar posibles vectores de ataque.

### Enumeración de puertos

```
pyrhos@debian-laptop:~$ sudo nmap -p- --open --min-rate 5000 -vvv -sS -n -Pn 10.10.11.92 -oN open-ports.txt
[sudo] contraseña para pyrhos: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-22 15:02 CET
Initiating SYN Stealth Scan at 15:02
Scanning 10.10.11.92 [65535 ports]
Discovered open port 22/tcp on 10.10.11.92
Discovered open port 80/tcp on 10.10.11.92
Discovered open port 5555/tcp on 10.10.11.92
Completed SYN Stealth Scan at 15:02, 20.60s elapsed (65535 total ports)
Nmap scan report for 10.10.11.92
Host is up, received user-set (0.12s latency).
Scanned at 2025-11-22 15:02:04 CET for 20s
Not shown: 63530 closed tcp ports (reset), 2002 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
5555/tcp open  freeciv syn-ack ttl 63


```

### Enumeración de los servicios

```
pyrhos@debian-laptop:~$ sudo nmap -p22,80,5555 -sCV 10.10.11.92 -oN portscan.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-22 15:04 CET
Nmap scan report for conversor.htb (10.10.11.92)
Host is up (0.11s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp   open  http     Apache httpd 2.4.52
| http-title: Login
|_Requested resource was /login
|_http-server-header: Apache/2.4.52 (Ubuntu)
5555/tcp open  freeciv?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 102.14 seconds

```
