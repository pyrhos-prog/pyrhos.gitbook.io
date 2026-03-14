---
icon: database
---

# Inyecciones

Una inyección ocurre cuando datos no confiables son enviados a un intérprete como parte de un comando o consulta. El intérprete ejecuta los datos como si fueran instrucciones legítimas, permitiendo al atacante modificar la lógica del comando original.

```
Principio universal de inyección:

App construye un comando/consulta concatenando input del usuario:
comando = "base_comando " + INPUT_USUARIO

Si INPUT_USUARIO contiene metacaracteres del intérprete:
INPUT = "normal; comando_malicioso"
→ Resultado: "base_comando normal; comando_malicioso"
→ El intérprete ejecuta ambos comandos
```

### Mapa de inyecciones (excluyendo SQLi y XSS)

| Tipo                       | Intérprete afectado                               | Impacto máximo                              |
| -------------------------- | ------------------------------------------------- | ------------------------------------------- |
| **Command Injection**      | Shell del SO (bash, cmd)                          | RCE como el proceso del servidor            |
| **SSTI**                   | Motor de plantillas (Jinja2, Twig, Freemarker...) | RCE en el servidor                          |
| **XXE**                    | Parser XML                                        | SSRF, lectura de archivos, RCE (raro)       |
| **SSRF**                   | Cliente HTTP del servidor                         | Acceso a red interna, metadata cloud        |
| **LDAP Injection**         | Servidor LDAP/AD                                  | Bypass de auth, data dump                   |
| **NoSQL Injection**        | MongoDB, Redis, CouchDB                           | Bypass de auth, data dump                   |
| **XPath Injection**        | Parser XML/XPath                                  | Extracción de datos XML                     |
| **CRLF Injection**         | Parser HTTP, logs                                 | Session splitting, log injection            |
| **Host Header Injection**  | Cabecera HTTP Host                                | SSRF, reset de contraseñas, cache poisoning |
| **HTTP Request Smuggling** | Proxy/servidor HTTP                               | Bypass de seguridad, ATO                    |

### ¿Por qué ocurren las inyecciones?

```
1. Concatenación directa de input del usuario en comandos/consultas
2. Falta de validación o sanitización del input
3. Uso de APIs inseguras cuando existen alternativas seguras
4. Confianza excesiva en datos de cabeceras HTTP
5. Deserialización de datos no confiables
```

### Patrón de detección universal

Independientemente del tipo de inyección, el proceso de detección es similar:

```
1. Encontrar puntos donde la app procesa input
   → Parámetros GET/POST, cabeceras, cookies, rutas URL, campos de formulario

2. Identificar el contexto del intérprete
   → ¿Qué tecnología procesa este input? (shell, template engine, XML parser...)

3. Inyectar metacaracteres del intérprete y observar cambios en la respuesta
   → Error visible → inyección confirmada
   → Cambio de comportamiento → inyección blind (inferencia)
   → Sin cambio aparente → probar técnicas out-of-band

4. Extraer información o ejecutar comandos
   → Explotación directa (resultados visibles)
   → Explotación blind (time-based o OOB)
```

### Metacaracteres por tipo de intérprete

```bash
# Shell (OS Command Injection)
;   &&   ||   |   `cmd`   $(cmd)   \n   %0a

# Template engines
{{7*7}}   ${7*7}   <%= 7*7 %>   #{7*7}   *{7*7}

# XML / XXE
<!DOCTYPE   <!ENTITY   &entity;   ]]>   <?xml

# LDAP
*   (   )   \   NUL   &   |

# NoSQL (MongoDB)
$where   $gt   $ne   $regex   {"$gt":""}   ;   //

# XPath
'   "   /   //   *   [   ]   @   .   ..   |

# HTTP Headers (CRLF)
\r\n   %0d%0a   \n   %0a
```

### Impacto según el tipo

```
Critical (RCE directo):
→ Command Injection
→ SSTI en motores sin sandboxing
→ XXE con expect:// o en contextos especiales

High (acceso a datos / red interna):
→ XXE (lectura de archivos, SSRF)
→ SSRF (acceso a metadatos cloud, red interna)
→ LDAP Injection (dump de AD)
→ NoSQL Injection (dump de base de datos)

Medium:
→ XPath Injection (extracción de datos XML)
→ CRLF Injection (inyección de headers/logs)
→ Host Header Injection (password reset, cache poisoning)
```

### Posición en OWASP Top 10

```
OWASP Top 10 — 2021:
#3 Injection
   → SQL Injection            - esta sección
   → XSS                      - esta sección
   → Command Injection        - esta sección
   → SSTI                     - esta sección
   → LDAP Injection           - esta sección
   → NoSQL Injection          - esta sección
   → XPath Injection          - esta sección

#10 Server-Side Request Forgery (SSRF)
   → SSRF                     → esta sección

Relacionado pero en otras categorías:
#4 Insecure Design → XXE, CRLF, Host Header
```

> Command Injection y SSTI son las inyecciones más críticas después de SQLi: ambas pueden dar RCE directo en el servidor con una sola petición.

> La detección de inyecciones empieza siempre por los metacaracteres del intérprete. Antes de lanzar payloads complejos, probar con los más simples: `; id`, `{{7*7}}`, `' OR '1'='1`.

> Muchas inyecciones son a ciegas: la app no muestra el resultado en la respuesta. Siempre tener preparadas técnicas de inferencia (time-based) y out-of-band (DNS, HTTP a servidor controlado).
