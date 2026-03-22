# Nessus — Escaneos y Configuración Avanzada

### Tipos de escaneo

Nessus ofrece plantillas predefinidas para distintos escenarios. Al crear un nuevo escaneo, se elige una plantilla como punto de partida.

#### Plantillas principales

| Plantilla                    | Cuándo usarla                                                                         |
| ---------------------------- | ------------------------------------------------------------------------------------- |
| **Basic Network Scan**       | Escaneo general de hosts en una red. La más usada para empezar.                       |
| **Advanced Scan**            | Control total sobre qué plugins se ejecutan. Para usuarios experimentados.            |
| **Advanced Dynamic Scan**    | Como Advanced pero los plugins se seleccionan dinámicamente según lo que se descubra. |
| **Credentialed Patch Audit** | Con credenciales del sistema — detecta software sin parchear localmente.              |
| **Web Application Tests**    | Orientado a aplicaciones web: XSS, SQLi, misconfiguraciones.                          |
| **Malware Scan**             | Detecta indicadores de malware en sistemas Windows/Linux.                             |
| **Spectre and Meltdown**     | Vulnerabilidades específicas de CPU (CVE-2017-5753 y derivados).                      |
| **PCI DSS**                  | Cumplimiento normativo PCI — para entornos de pago con tarjeta.                       |
| **DISA STIG**                | Cumplimiento de las guías de seguridad del DoD (entornos gubernamentales).            |

#### Escaneo con y sin credenciales

Un escaneo **sin credenciales** (unauthenticated) detecta vulnerabilidades visibles desde la red: puertos abiertos, versiones de servicios expuestos, certificados, configuraciones accesibles remotamente.

Un escaneo **con credenciales** (authenticated) accede al sistema con usuario y contraseña y puede detectar: software instalado sin parchear, configuraciones incorrectas del SO, cuentas con contraseñas débiles, permisos de archivos inseguros y mucho más. Es significativamente más completo.

### Crear un escaneo básico

1. New Scan → Basic Network Scan
2. Rellenar los campos principales:

| Campo        | Descripción                                                                                       |
| ------------ | ------------------------------------------------------------------------------------------------- |
| **Name**     | Nombre descriptivo del escaneo                                                                    |
| **Targets**  | IPs, rangos CIDR o nombres de host. Ejemplos: `192.168.1.1`, `192.168.1.0/24`, `host.ejemplo.com` |
| **Schedule** | Inmediato o programado (fecha/hora específica)                                                    |

3. Guardar y hacer clic en el botón de lanzar (triángulo).

### Configuración avanzada de escaneos

La pestaña **Settings** de cada escaneo contiene múltiples secciones de configuración:

#### Discovery

Controla cómo Nessus descubre hosts y puertos.

```
Host Discovery:
→ Ping methods: ICMP, TCP SYN, ARP
→ Considerar hosts activos aunque no respondan al ping

Port Scanning:
→ Port range: "default" (top 4.789 puertos) / "all" (65.535) / rango personalizado
→ Scan technique: SYN (más rápido y sigiloso) / TCP Connect (más ruidoso)
→ Local Port Enumerators: usar netstat si hay credenciales
```

#### Assessment

Define qué tipo de comprobaciones de vulnerabilidades se realizan.

```
General Settings:
→ Safe Checks: ON (desactivar checks que pueden crashear sistemas)
  → Mantener activado en sistemas de producción
→ Override normal accuracy: ajustar agresividad de la detección

Brute Force:
→ Hydra puede intentar credenciales por defecto en servicios como FTP, SSH, Telnet
→ Solo habilitar con permiso explícito
```

#### Report

Configura el formato y contenido de los informes generados.

```
→ Processing: show missing patches, override severity de plugins específicos
→ Output: mostrar resultados sobreescribibles, hosts que no responden
→ Show hosts that were not scanned
```

#### Advanced

```
→ Max concurrent checks per host: limitar la carga sobre un host (defecto: 5)
→ Max concurrent hosts per scan: cuántos hosts escanear en paralelo
→ Network Timeout: tiempo de espera para respuestas de red
→ Throttle scan when CPU usage is high: reducir velocidad si el escáner está saturado
```

### Escaneo con credenciales

Con credenciales, Nessus accede al sistema y puede auditar el software instalado, los parches pendientes y la configuración interna.

Configuración en la pestaña **Credentials** del escaneo:

#### SSH (Linux/Unix)

```
Authentication method:
→ password: usuario + contraseña directamente
→ public key: clave privada SSH (.pem o pegada en texto)
→ certificate: autenticación por certificado

Elevate privileges:
→ sudo: ejecutar comprobaciones privilegiadas con sudo
→ su: cambiar a root con contraseña
→ Sudoers file: especificar archivo sudoers custom
```

#### Windows (SMB)

```
→ Username / Password / Domain
→ LM Hash / NTLM Hash (para PtH si se dispone del hash)
→ Kerberos (entornos de dominio)

Consideraciones:
→ El usuario debe tener permisos de administrador local
→ Puede requerir habilitar el registro remoto (RemoteRegistry service)
→ La cuenta Invitado debe estar deshabilitada
```

#### Bases de datos

Nessus también puede conectarse directamente a bases de datos para auditarlas:

* Oracle, MySQL, MSSQL, PostgreSQL, MongoDB
* Cada uno requiere usuario, contraseña, puerto y base de datos destino

### Políticas personalizadas

Las **Policies** permiten guardar una configuración de escaneo completa y reutilizarla. Útil cuando se realizan escaneos periódicos del mismo entorno.

Para crear una política: Policies → New Policy → seleccionar plantilla base → configurar → guardar. Al crear nuevos escaneos, se puede seleccionar la política en lugar de la plantilla.

### Plugin Rules

Permiten modificar la severidad que Nessus asigna a un plugin específico en un host concreto. Útil cuando:

* Un hallazgo es un falso positivo conocido → se puede reducir a Informational
* Un hallazgo tiene mayor impacto en el entorno específico → se puede elevar la severidad
* Se quiere ocultar un hallazgo aceptado del riesgo del informe

Configuración en Settings → Plugin Rules → Add Rule.

> Para entornos de producción, activar siempre **Safe Checks**. Algunos plugins pueden provocar inestabilidad o reinicio de servicios si se ejecutan en sistemas frágiles.

> Los escaneos con credenciales son mucho más completos pero también más visibles en los logs del sistema. Coordinar siempre con el equipo de sistemas antes de ejecutarlos en entornos de producción.
