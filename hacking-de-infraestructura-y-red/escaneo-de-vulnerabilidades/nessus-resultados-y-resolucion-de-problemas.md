# Nessus — Resultados y Resolución de Problemas

### Trabajar con los resultados del escaneo

Una vez finalizado el escaneo, los resultados se muestran organizados por host y por severidad.

#### Niveles de severidad

| Nivel        | Color       | CVSS aproximado | Descripción                                                       |
| ------------ | ----------- | --------------- | ----------------------------------------------------------------- |
| **Critical** | Rojo oscuro | 9.0 – 10.0      | Explotación remota sin autenticación, impacto total               |
| **High**     | Rojo        | 7.0 – 8.9       | Vulnerabilidades graves con alta probabilidad de explotación      |
| **Medium**   | Naranja     | 4.0 – 6.9       | Requieren condiciones específicas o acceso previo                 |
| **Low**      | Amarillo    | 0.1 – 3.9       | Impacto limitado, difíciles de explotar en aislamiento            |
| **Info**     | Azul        | 0.0             | No son vulnerabilidades — son datos de configuración o inventario |

#### Navegar los resultados

La vista de resultados tiene dos perspectivas:

* **By Host** — ver todos los hallazgos de cada sistema individual. Útil para priorizar qué sistema remediar primero.
* **By Plugin** — ver todos los sistemas afectados por un mismo CVE. Útil para entender la extensión de una vulnerabilidad concreta.

Al hacer clic en un hallazgo concreto se muestra:

```
Plugin Information:
→ Plugin ID, familia de plugins, severidad CVSS v2 y v3

Description:
→ Explicación de la vulnerabilidad, cómo funciona y por qué es un riesgo

Solution:
→ Acción de remediación recomendada (parche, configuración, versión a actualizar)

See Also:
→ Referencias a CVE, CVSS, advisories oficiales, artículos técnicos

Output:
→ Evidencia específica encontrada en el host: versión detectada, respuesta del servicio...

Risk Information:
→ CVSS Base Score, Vector CVSS, CVE IDs relacionados
```

### Priorización de hallazgos

No todos los Critical son igual de urgentes. Para priorizar correctamente hay que cruzar la severidad del CVE con el contexto del entorno:

**Factores que elevan la prioridad:**

* El host está expuesto a internet directamente
* La vulnerabilidad tiene exploit público disponible (Exploit-DB, Metasploit)
* El servicio vulnerable procesa datos sensibles
* La vulnerabilidad permite RCE sin autenticación

**Factores que reducen la prioridad:**

* El host solo es accesible desde la red interna
* El servicio no está en uso real
* Hay controles compensatorios (WAF, firewall, segmentación)
* La vulnerabilidad requiere acceso previo para explotarse

### Exportar resultados

Nessus permite exportar los resultados en varios formatos para incluir en informes.

Desde la pantalla de resultados → Export:

| Formato              | Uso                                                                            |
| -------------------- | ------------------------------------------------------------------------------ |
| **Nessus (.nessus)** | Formato nativo XML para reimportar en Nessus o procesar con otras herramientas |
| **PDF**              | Informe visual para clientes y dirección. Configurable (ejecutivo o técnico)   |
| **CSV**              | Tabla de todos los hallazgos para análisis en Excel/Python                     |
| **HTML**             | Informe web navegable                                                          |

```bash
# Procesar el CSV con Python para análisis personalizado
import pandas as pd

df = pd.read_csv('nessus_results.csv')

# Ver solo Criticals y Highs
criticos = df[df['Risk'].isin(['Critical', 'High'])]

# Agrupar por host
por_host = df.groupby('Host')['Risk'].value_counts().unstack(fill_value=0)

# Exportar solo los CVEs únicos afectados
df[['CVE', 'Plugin Name', 'Risk']].drop_duplicates().to_csv('cves_unicos.csv', index=False)
```

### Filtros y búsquedas en resultados

La barra de filtros permite reducir los resultados a lo relevante:

```
Filtros disponibles:
→ Severity: seleccionar uno o varios niveles
→ Plugin Family: web servers, databases, Windows, etc.
→ Plugin Name: buscar por nombre de vulnerabilidad o CVE
→ Host: filtrar por IP específica
→ Port: ver solo hallazgos en un puerto concreto
→ VPR Score (Vulnerability Priority Rating): puntuación de riesgo real de Tenable
```

El **VPR Score** de Tenable es más útil que el CVSS clásico porque incorpora factores adicionales como si hay exploit activo en la naturaleza, si está siendo explotado en campañas recientes y la madurez del exploit.

### Comparar escaneos (Scan Comparison)

Nessus Pro permite comparar dos escaneos del mismo objetivo para ver:

* Vulnerabilidades **nuevas** (aparecieron entre el escaneo anterior y el actual)
* Vulnerabilidades **resueltas** (existían antes y ya no)
* Vulnerabilidades **persistentes** (siguen sin remediar)

Muy útil para demostrar el progreso de remediación entre auditorías.

### Problemas comunes y solución

#### El escaneo no detecta hosts

```
Causa más común: el firewall del host bloquea el ping (ICMP)
→ En Discovery → Host Discovery: habilitar TCP SYN ping además de ICMP
→ Probar con TCP SYN en puertos 22, 80, 443

Causa: Nessus está en otra VLAN sin acceso al rango escaneado
→ Verificar conectividad desde el host de Nessus antes de escanear
→ ping / traceroute desde el servidor de Nessus al target

Causa: el rango de IPs introducido es incorrecto
→ Verificar la notación CIDR: 192.168.1.0/24 (no 192.168.1.1/24)
```

#### El escaneo detecta muy pocas vulnerabilidades

```
Causa: escaneo sin credenciales en un sistema con firewall
→ Añadir credenciales → diferencia enorme en resultados
→ Los escaneos sin credenciales solo ven la superficie expuesta en red

Causa: los plugins no están actualizados
→ Settings → Software Update → Update Plugins

Causa: el rango de puertos es demasiado limitado
→ En Discovery → Port Scanning: cambiar "default" a "all"
→ Los servicios en puertos no estándar no se detectan con el rango default
```

#### Fallos de autenticación en escaneos con credenciales (Windows)

```
Error: "Authentication Failure"
→ Verificar usuario/contraseña o hash NTLM
→ El usuario debe ser administrador local
→ Habilitar el servicio RemoteRegistry en el host:
  services.msc → Remote Registry → Automatic → Start

Error: acceso denegado aunque las credenciales son correctas
→ Comprobar que el Firewall de Windows permite el acceso SMB entrante
→ Verificar que la cuenta no está bloqueada (Account Lockout)
→ En workgroups (no dominio): puede requerir deshabilitar UAC remoto:
  reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System"
      /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

#### Fallos de autenticación SSH (Linux)

```
→ Verificar que el usuario puede hacer ssh desde el mismo host de Nessus
→ Comprobar que SSH permite el método de autenticación configurado
  (cat /etc/ssh/sshd_config → PasswordAuthentication yes/no)
→ Para sudo: verificar que el usuario está en sudoers sin contraseña para los comandos necesarios
→ Comprobar que no hay rate limiting en el SSH del target (fail2ban)
```

#### El escaneo tarda demasiado

```
→ Reducir Max Concurrent Checks per Host (de 5 a 3)
→ Reducir Max Concurrent Hosts (escanear menos hosts en paralelo)
→ Limitar el Port Range si no se necesitan todos los puertos
→ Usar Safe Checks para evitar comprobaciones lentas y pesadas
→ Verificar el ancho de banda disponible entre el escáner y el objetivo
```

#### Falsos positivos frecuentes

```
Causa: Nessus detecta una versión del banner sin verificar si está parcheada
→ Muchas distribuciones Linux (RHEL, Debian) aplican parches sin cambiar el número de versión
→ La solución real: escanear con credenciales (Nessus puede leer el changelog del paquete)
→ Alternativa: crear un Plugin Rule que reduzca la severidad del hallazgo

Causa: el servicio está en mantenimiento y no responde correctamente
→ Repetir el escaneo cuando el servicio esté operativo
```

### Buenas prácticas

* Escanear siempre **primero sin credenciales** para ver qué expone el sistema a la red, y luego **con credenciales** para el análisis interno completo.
* Guardar todos los escaneos en formato `.nessus` para referencia histórica y comparación futura.
* Documentar los falsos positivos confirmados con Plugin Rules para que no reaparezcan en escaneos periódicos.
* En entornos de producción, programar los escaneos fuera del horario de negocio y con Safe Checks activo.
* Combinar los resultados de Nessus con verificación manual — especialmente en hallazgos Critical antes de reportarlos.

> Los hallazgos **Info** no son vulnerabilidades pero contienen información valiosa: inventario de software, configuraciones, versiones, certificados SSL y puertos abiertos. Revisarlos siempre — a veces revelan servicios olvidados o configuraciones inesperadas.

> Un resultado de Nessus no es un informe de pentest. Nessus detecta vulnerabilidades conocidas pero no encadena ataques, no demuestra impacto real ni identifica lógica de negocio vulnerable. El valor añadido del pentester está en interpretar y explotar los resultados, no en generarlos.
