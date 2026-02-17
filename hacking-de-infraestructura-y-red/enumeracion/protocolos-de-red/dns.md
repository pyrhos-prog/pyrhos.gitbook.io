---
icon: building-magnifying-glass
---

# DNS

DNS (Domain Name System) resuelve nombres de dominio a direcciones IP.\
Sistema distribuido y jerárquico.\
**Protocolo principal:**&#x20;

* UDP :53&#x20;
* TCP :53 para transferencias de zona y respuestas grandes

No está cifrado por defecto.

### Jerarquía

Root (.)\
→ TLD (.com, .net, .org…)\
→ Dominio de segundo nivel (empresa.com)\
→ Subdominios (dev.empresa.com)\
→ Host (ws01.dev.empresa.com)

### Tipos de Servidores

* **Root Server** → Gestiona TLD
* **Authoritative** → Tiene autoridad sobre una zona
* **Recursive / Resolver** → Resuelve consultas para el cliente
* **Caching Server** → Guarda respuestas temporalmente
* **Forwarder** → Reenvía consultas a otro DNS

### Registros DNS

| Registro | Función                                           |
| -------- | ------------------------------------------------- |
| A        | Dominio → IPv4                                    |
| AAAA     | Dominio → IPv6                                    |
| MX       | Servidor de correo                                |
| NS       | Nameservers de la zona                            |
| TXT      | Información arbitraria (SPF, DMARC, verificación) |
| CNAME    | Alias de otro dominio                             |
| PTR      | IP → Dominio (reverse lookup)                     |
| SOA      | Información principal de la zona                  |

### Transferencia de Zona (AXFR)

Una transferencia de zona es el proceso mediante el cual un servidor DNS copia todos los registros de una zona desde otro servidor.

* Usa TCP/53
* Sincroniza zona entre master y slave
* Si `allow-transfer` está mal configurado → exposición completa de la zona

**Comando**:

```bash
dig axfr dominio.com @IP_DNS
```

| Parámetro        | Función                                                                                                   |
| ---------------- | --------------------------------------------------------------------------------------------------------- |
| `axfr`           | Tipo de consulta: _Asynchronous Full Transfer Zone_, solicita la zona completa                            |
| `dominio.com`    | Nombre de la zona que queremos transferir                                                                 |
| `@IP_DEL_DNS`    | Especifica el servidor DNS al que se le envía la consulta                                                 |
| `+tcp`           | Fuerza la consulta sobre TCP (opcional, recomendado para AXFR grandes)                                    |
| `+noall +answer` | Opciones para mostrar solo la respuesta y no toda la información del query (útil para limpieza de salida) |

## Enumeración con Dig

### Consultas Básicas

#### Resolver dominio

```bash
dig dominio.com
```

#### Especificar servidor DNS

```bash
dig dominio.com @IP_DNS
```

#### Forzar TCP

```bash
dig dominio.com +tcp
```

#### Mostrar solo respuesta limpia

```bash
dig dominio.com +noall +answer
```

### Enumeración de Registros

#### A (IPv4)

```bash
dig dominio.com A
```

#### AAAA (IPv6)

```bash
dig dominio.com AAAA
```

#### NS (Nameservers)

```bash
dig dominio.com NS
```

#### MX (Mail Servers)

```bash
dig dominio.com MX
```

#### TXT (SPF / DKIM / DMARC / Verificación)

```bash
dig dominio.com TXT
```

#### SOA (Start of Authority)

```bash
dig dominio.com SOA
```

#### ANY (Todos los registros expuestos)

```bash
dig dominio.com ANY @IP_DNS
```

### Reverse Lookup

#### IP → Dominio

```bash
dig -x IP
```

### Información del Servidor DNS

#### Intentar obtener versión de BIND

```bash
dig CH TXT version.bind @IP_DNS
```

### Transferencia de Zona (AXFR)

#### Transferencia completa

```bash
dig axfr dominio.com @IP_DNS
```

#### Forzar TCP explícitamente

```bash
dig axfr dominio.com @IP_DNS +tcp
```

Si funciona → devuelve todos los registros de la zona.

### Fuerza Bruta Manual de Subdominios

```bash
for sub in $(cat wordlist.txt); do
  dig $sub.dominio.com @IP_DNS +short
done
```

### Interpretación de Salida

Formato general:

nombre TTL IN TIPO valor

* nombre → subdominio/host
* TTL → tiempo de cache
* IN → clase Internet
* TIPO → A, MX, NS, etc.
* valor → IP, servidor, alias

### Flujo de Enumeración Recomendado

1. dig dominio.com NS
2. dig dominio.com MX
3. dig dominio.com TXT
4. dig dominio.com SOA
5. dig axfr dominio.com @NS
6. Reverse lookup sobre IPs encontradas
7. Fuerza bruta de subdominios

Objetivo:

* Mapear infraestructura
* Detectar AXFR abierto
* Identificar hosts internos
* Descubrir servicios críticos
