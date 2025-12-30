# Puerto 53 (DNS)

### **Enumeración con Nmap**

{% code expandable="true" %}
```
pyrhos@debian:~$ sudo nmap --script=*dns* x.com
o
pyrhos@debian:~$ nmap --script=dns-nsec-enum,dns-service-discovery,dns-zone-transfer,
dns-srv-enum,dns-brute,dns-recursion,dns-nsec3-enum,dns-zeustracker <ip-víctima>
```
{% endcode %}

### **Enumeración con dnslookup**

```
pyrhos@debian:~$ dnslookup x.com
dnslookup 1.11.1-11969
Server: 100.64.0.23:53

dnslookup result (elapsed 93.827742ms):
;; opcode: QUERY, status: NOERROR, id: 35632
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;x.com. IN       A

;; ANSWER SECTION:
x.com.  205     IN      A       172.66.0.227
```

### DNS Zone Transfer

```
dig AXFR x.com @servidor_dns
```
