# Enumeracion de subdominios

La enumeración de subdominios es una parte crucial de la enumeración web, esta parte de la enumeración te puede dar acceso a subdominios cruciales como:

* apartados en desarrollo
* paneles de login&#x20;
* administración

Podemos realizar una enumeración de subdominios de manera pasiva o activa.

## Enumeración pasiva

### Google Dorks

Se pueden llegar a encontrar subdominios a través de búsquedas avanzadas "Google Dorks", sin tener una interacción directa con el objetivo.

**Forma manual:**

| Dork                                | Descripción                      |
| ----------------------------------- | -------------------------------- |
| `site:example.com`                  | Resultados indexados del dominio |
| `site:*.example.com`                | Subdominios indexados por Google |
| `site:example.com -www.example.com` | Excluye el dominio principal     |
| `site:example.com inurl:dev`        | Subdominios de desarrollo        |
| `site:example.com inurl:test`       | Subdominios de pruebas           |
| `site:example.com inurl:beta`       | Subdominios beta                 |
| `site:example.com inurl:api`        | Subdominios de API               |
| `site:example.com inurl:admin`      | Subdominios administrativos      |
| `site:example.com inurl:portal`     | Subdominios tipo portal          |
| `site:example.com inurl:internal`   | Subdominios internos             |
| `site:example.com inurl:intranet`   | Subdominios de intranet          |
| `site:example.com inurl:corp`       | Subdominios corporativos         |
| `site:example.com inurl:services`   | Subdominios de servicios         |
| `site:example.com inurl:apps`       | Subdominios de aplicaciones      |
| `site:example.com inurl:cloud`      | Subdominios en la nube           |

**Forma automatizada:**

Existen paginas web que generan Dorks para diferentes tipos de busquedas. Estos son algunos ejemplos:

{% embed url="https://www.dorkgpt.com/" %}

{% embed url="https://dorksearch.com/" %}

{% embed url="https://dorkgenius.com/" %}

### Subfinder

Subfinder es una herramienta que enumera subdominios de forma pasiva, sin interactuar con la web objetivo.

**Repositorio de la herramienta:**

{% embed url="https://github.com/projectdiscovery/subfinder?tab=readme-ov-file" %}

**Guia:**

{% embed url="https://dhiyaneshgeek.github.io/bug/bounty/2020/02/06/recon-with-me/" %}



## Enumeración activa

La enumeración activa interactúa directamente con el objetivo, por lo que es mas fácil que nos detecten pero también podemos encontrar mas cosas que con una enumeración pasiva.

### Gobuster

Gobuster permite realizar fuerza bruta a subdominios utilizando diccionarios como los de SecList.

**Ejemplo de uso:**

```
pyrhos$ gobuster dns -d example.com -w subdomains.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial)
[+] Domain: example.com
[+] Threads: 10
[+] Wordlist: subdomains.txt
[+] Timeout: 1s
Found: admin.example.com
Found: dev.example.com
Found: api.example.com
Found: staging.example.com
```

### FFuF

#### Enumeración de subdominios (método normal)

```
ffuf -w subdomains.txt -u http://FUZZ.example.com

    /'___\  /'___\           /'___\
   /\ \__/ /\ \__/  __  __  /\ \__/
   \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
    \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
     \ \_\    \ \_\  \ \____/  \ \_\
      \/_/     \/_/   \/___/    \/_/


dev [Status: 200, Size: 15432]
admin [Status: 403, Size: 532]
api [Status: 200, Size: 9821]
internal [Status: 301, Size: 0]
staging [Status: 200, Size: 16784]
```

#### Enumeración de subdominios (Host Header)

Este método es el más utilizado cuando todos los subdominios resuelven a la misma IP (vhosts).

```
ffuf -w subdomains.txt -u https://example.com
 -H "Host: FUZZ.example.com"

    /'___\  /'___\           /'___\
   /\ \__/ /\ \__/  __  __  /\ \__/
   \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
    \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
     \ \_\    \ \_\  \ \____/  \ \_\
      \/_/     \/_/   \/___/    \/_/


dev [Status: 200, Size: 15432]
admin [Status: 403, Size: 532]
api [Status: 200, Size: 9821]
internal [Status: 301, Size: 0]
staging [Status: 200, Size: 16784]
```

* `FUZZ` se sustituye por cada entrada del diccionario.
* El encabezado `Host` permite identificar subdominios virtuales en el mismo servidor.
* Este método es especialmente efectivo en servidores con virtual hosting.
* La salida mostrada es simulada y solo tiene fines educativos.
