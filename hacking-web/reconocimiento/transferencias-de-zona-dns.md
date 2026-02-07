---
icon: spider-web
---

# Transferencias de zona DNS

Una transferencia de zona DNS copia **todos los registros de un dominio** desde un servidor DNS a otro. Si está mal configurada, permite a terceros descargar la zona completa.

### Por qué es crítica

Expone la estructura completa del dominio sin fuerza bruta.

### Informacion que revela

* Subdominios internos y no públicos
* Direcciones IP asociadas
* Servidores DNS autorizados
* Servicios definidos en DNS (MX, SRV, etc.)

En Active Directory puede revelar:

* Domain Controllers
* Servidores críticos
* Convenciones de nombres internas

### Flujo básico

1. Solicitud AXFR al servidor DNS
2. Envío del registro SOA
3. Transferencia de todos los registros
4. Finalización de la zona

### Explotación con dig

Solicitar transferencia de zona:

```bash
dig axfr @<DNS_SERVER>
```

Ejemplo:

```bash
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

Si está permitido, devuelve todos los registros DNS del dominio.

### Impacto

* Mapeo completo del dominio
* Enumeración rápida de objetivos
* Base para escaneo y ataques posteriores
