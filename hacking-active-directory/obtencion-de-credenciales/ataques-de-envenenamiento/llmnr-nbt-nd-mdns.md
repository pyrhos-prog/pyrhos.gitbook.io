# LLMNR - NBT-ND - mDNS

> Fase: Credential Harvesting / Network Poisoning — punto de apoyo inicial en AD Sigue a: Enumeración inicial del dominio (usuarios, grupos, hosts, DC) Precede a: Password Spraying · SMB Relay (módulo de movimiento lateral)

### Objetivo de la fase

Conseguir credenciales válidas (texto claro o hash) de una cuenta de dominio mediante envenenamiento de protocolos de resolución de nombres, en paralelo a password spraying. El resultado es un primer punto de apoyo acreditado en el dominio.

***

### Fundamentos: LLMNR y NBT-NS

Mecanismos de resolución de nombres alternativos a DNS en entornos Windows.

| Protocolo                                    | Función                                                      | Puerto   |
| -------------------------------------------- | ------------------------------------------------------------ | -------- |
| LLMNR (Link-Local Multicast Name Resolution) | Resolución vía multicast cuando falla DNS                    | UDP 5355 |
| NBT-NS (NetBIOS Name Service)                | Identifica hosts por nombre NetBIOS; fallback si falla LLMNR | UDP 137  |

Flujo normal de resolución:

1. Host pide resolver un nombre → falla DNS.
2. Lanza consulta LLMNR en broadcast a la red local.
3. Si nadie responde, cae a NBT-NS.

Fallo de diseño explotable: cualquier host de la red puede responder a estas consultas. No hay autenticación de quién contesta.

***

### TTP / flujo de ataque

Responder se hace pasar por el host que la víctima busca y responde primero. Si la víctima intenta autenticarse contra ese "host", se captura su hash NetNTLMv1/v2.

Ejemplo de flujo:

1. Un host intenta conectar a `\\print01.dominio.local` pero escribe mal `\\printer01.dominio.local`.
2. DNS responde: nombre desconocido.
3. El host pregunta por broadcast en la red local quién es `printer01`.
4. Responder contesta: "soy yo".
5. El host envía usuario + hash NTLMv2 al atacante creyendo que es el recurso legítimo.
6. El hash se craquea offline o se reenvía en un ataque SMB Relay (si SMB signing está deshabilitado).

Nota: LLMNR/NBT-NS spoofing + falta de SMB signing es una de las rutas más directas a Domain Admin en evaluaciones internas mal asseguradas. SMB Relay se cubre en el módulo de movimiento lateral — aquí solo se captura el hash.

***

### Herramientas

| Herramienta | Notas                                                   |
| ----------- | ------------------------------------------------------- |
| Responder   | Python, estándar en Linux. Existe `.exe` para Windows.  |
| Inveigh     | C# / PowerShell (legacy). MITM multiplataforma.         |
| Metasploit  | Módulos de scanner/spoofing integrados, menos flexible. |

Protocolos atacables por ambas: `LLMNR, DNS, MDNS, NBNS, DHCP, ICMP, HTTP, HTTPS, SMB, LDAP, WebDAV, Proxy Auth`.

Responder además soporta: `MSSQL, DCE-RPC, FTP/POP3/IMAP/SMTP auth`.

***

### Comandos

Modo análisis (pasivo, no envenena — solo escucha):

```bash
responder -I eth0 -A
```

Modo activo (envenenando):

```bash
sudo responder -I ens224
```

Combinación típica para trabajo real (WPAD rogue + fingerprinting):

```bash
sudo responder -I eth0 -wf
```

#### Flags

| Flag   | Función                                                                                  |
| ------ | ---------------------------------------------------------------------------------------- |
| `-I`   | Interfaz (o `ALL`)                                                                       |
| `-A`   | Modo análisis, no responde                                                               |
| `-w`   | Levanta servidor rogue WPAD                                                              |
| `-f`   | Fingerprinting del SO del host que consulta                                              |
| `-v`   | Verbose, mucho ruido                                                                     |
| `-F`   | Fuerza auth NTLM/Basic en retrieval de WPAD — puede generar prompt visible               |
| `-P`   | Fuerza auth de proxy (NTLM transparente / Basic con prompt), efectivo combinado con `-r` |
| `-r`   | Responde a sufijo wredir — puede romper cosas en la red, off por defecto                 |
| `-d`   | Responde a sufijo de dominio NBT-NS — puede romper cosas, off por defecto                |
| `--lm` | Downgrade a hash LM (XP/2003 y anteriores)                                               |

Requisitos:

* Ejecutar como root/sudo.
* Puertos libres en el host atacante:

```
UDP 137, 138, 53, 1434
TCP/UDP 389
TCP 1433, 80, 135, 139, 445, 21, 3141, 25, 110, 587, 3128
Multicast UDP 5355, 5353
```

***

### Logs y output

Ubicación por defecto: `/usr/share/responder/logs/`

Formato de archivo: `(MODULO)-(TIPO_HASH)-(IP_CLIENTE).txt`

Ejemplo: `SMB-NTLMv2-SSP-172.16.5.25.txt`

También se guarda todo en SQLite, configurable en `Responder.conf` (mismo directorio, salvo clon del repo desde GitHub).

Práctica recomendada: lanzar Responder en una sesión `tmux` en background mientras se sigue enumerando (BloodHound, SMB shares, etc.) para maximizar hashes capturados sin parar la evaluación.

***

### Crackeo offline

Los hashes NetNTLMv2 no son pass-the-hash compatibles — hay que crackearlos offline.

```bash
hashcat -m 5600 hash_ntlmv2.txt /usr/share/wordlists/rockyou.txt
```

* Modo Hashcat para NetNTLMv2: 5600
* Hash no identificable → consultar example-hashes de Hashcat para el modo correcto.
* John the Ripper es la alternativa directa a Hashcat para este mismo flujo.

Filtrar antes de craquear: no perder tiempo en cuentas que no aportan valor para la fase siguiente. Priorizar cuentas con privilegios o membresías interesantes detectadas en la enumeración previa.

***

### Resumen estratégico

* LLMNR/NBT-NS poisoning es de las primeras técnicas a lanzar en cualquier engagement interno desde posición sin credenciales — bajo coste, alto valor, se puede dejar corriendo en paralelo a otras fases.
* El impacto real no es solo craquear una contraseña débil: es el punto de apoyo para pivotar (SMB Relay si no hay signing, o reenumeración con BloodHound/NetExec usando credenciales válidas).
* Siguiente paso en la ruta de estudio: Inveigh (mismo objetivo, ecosistema Windows), luego SMB Relay con `ntlmrelayx` explotando la falta de SMB signing sin crackeo offline.

***

#### Referencias internas

* Ver: `responder-vs-inveigh.md` (pendiente)
* Ver: `ntlmrelayx-smb-relay.md` (ya construido en el lab de AD — Helpdesk -> it-admin GenericAll)
