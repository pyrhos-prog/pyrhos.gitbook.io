# OpenVAS / GVM — Introducción y Primeros Pasos

**OpenVAS** (Open Vulnerability Assessment Scanner) es el escáner de vulnerabilidades open source más completo disponible. Forma parte del framework **Greenbone Vulnerability Management (GVM)**, que es el nombre actual del proyecto completo. Es la alternativa gratuita a Nessus Professional y es ampliamente usado en laboratorios, entornos de formación y organizaciones con presupuesto limitado.

### OpenVAS vs Nessus

|                     | OpenVAS / GVM                       | Nessus                                       |
| ------------------- | ----------------------------------- | -------------------------------------------- |
| **Licencia**        | Open source (gratuito)              | Propietaria (Essentials gratis, Pro de pago) |
| **Feed de plugins** | Greenbone Community Feed (\~80.000) | Tenable Plugin Feed (+180.000)               |
| **Precisión**       | Buena, en mejora constante          | Alta, referencia del sector                  |
| **Interfaz**        | GSA — web funcional                 | Web moderna e intuitiva                      |
| **Instalación**     | Múltiples servicios, más compleja   | Paquete único sencillo                       |
| **Uso típico**      | Labs, entornos sin presupuesto      | Auditorías profesionales                     |
| **Comunidad**       | Activa, open source                 | Soporte oficial de Tenable                   |

### Arquitectura de GVM

GVM no es una sola aplicación sino un conjunto de componentes que trabajan juntos:

| Componente                                | Función                                                                     |
| ----------------------------------------- | --------------------------------------------------------------------------- |
| **GSA** (Greenbone Security Assistant)    | Interfaz web — panel de control accesible desde el navegador                |
| **GVM** (Greenbone Vulnerability Manager) | Backend — gestiona escaneos, tareas y resultados                            |
| **OpenVAS Scanner**                       | Motor de escaneo — ejecuta los NVTs contra los objetivos                    |
| **NVTs** (Network Vulnerability Tests)    | Los plugins individuales de detección (equivalente a los plugins de Nessus) |
| **PostgreSQL**                            | Base de datos donde se almacenan resultados y configuración                 |
| **GVM-CLI**                               | Interfaz de línea de comandos para automatización                           |

### Instalación en Kali Linux

Kali incluye GVM en sus repositorios, lo que simplifica mucho la instalación.

```bash
# Instalar GVM
sudo apt update && sudo apt install gvm -y

# Configuración inicial (descarga los feeds de NVTs — tarda bastante)
sudo gvm-setup

# El setup descarga automáticamente:
# → NVTs (plugins de detección)
# → SCAP data (CVEs y CPEs)
# → CERT data (advisories)
# Este proceso puede tardar entre 15 y 45 minutos en la primera ejecución

# Verificar que todo está correcto tras el setup
sudo gvm-check-setup

# Si hay errores, ejecutar de nuevo el setup
sudo gvm-setup

# Iniciar los servicios
sudo gvm-start

# Para acceder a la interfaz web
# → https://127.0.0.1:9392
# → El setup genera una contraseña aleatoria para el usuario admin
#    que se muestra al final del proceso — anotarla
```

#### Ver las credenciales generadas

```bash
# Si no se anotó la contraseña durante el setup
sudo cat /etc/gvm/setup-config | grep password

# Cambiar la contraseña del admin
sudo gvmd --user=admin --new-password=nuevacontraseña
```

### Acceso a la interfaz web (GSA)

Una vez iniciados los servicios, acceder a `https://127.0.0.1:9392` en el navegador. El certificado es autofirmado — aceptar la advertencia del navegador y continuar.

#### Panel principal

La interfaz GSA organiza sus funciones en el menú superior:

| Sección            | Descripción                                                |
| ------------------ | ---------------------------------------------------------- |
| **Dashboards**     | Vista general con gráficas de vulnerabilidades y actividad |
| **Scans**          | Gestión de tareas y resultados de escaneo                  |
| **Assets**         | Inventario de hosts y sistemas descubiertos                |
| **Resilience**     | Tickets de remediación y planes de acción                  |
| **SecInfo**        | Base de datos de NVTs, CVEs, CPEs y CERTs                  |
| **Configuration**  | Targets, credenciales, scan configs y alertas              |
| **Administration** | Usuarios, grupos, feeds y configuración del sistema        |

### Conceptos clave de GVM

Antes de crear el primer escaneo conviene entender los objetos con los que trabaja GVM:

| Objeto          | Descripción                                                                               |
| --------------- | ----------------------------------------------------------------------------------------- |
| **Target**      | Define qué hosts escanear: IPs, rangos CIDR, nombres de host y las credenciales a usar    |
| **Scan Config** | Define qué NVTs ejecutar y con qué configuración (equivalente a las plantillas de Nessus) |
| **Task**        | Combina un Target + una Scan Config para crear un trabajo de escaneo                      |
| **Report**      | Los resultados generados por una Task al ejecutarse                                       |
| **Credential**  | Credenciales SSH, SMB o SNMP para escaneos autenticados                                   |
| **Alert**       | Notificaciones automáticas por email o webhook cuando se detectan vulnerabilidades        |

### Actualizar los feeds de NVTs

Los feeds de vulnerabilidades deben estar actualizados para detectar las últimas CVEs. GVM actualiza automáticamente si tiene acceso a internet, pero también se puede forzar manualmente:

```bash
# Actualizar feeds manualmente
sudo greenbone-nvt-sync
sudo greenbone-scapdata-sync
sudo greenbone-certdata-sync

# O con gvm-cli
sudo gvm-cli socket --gmp-username admin --gmp-password contraseña \
    --xml "<sync_feed><type>NVT</type></sync_feed>"

# Verificar la fecha del último update en la interfaz:
# Administration → Feed Status
```

> Al igual que Nessus, OpenVAS necesita los feeds actualizados para detectar vulnerabilidades recientes. Si el feed tiene varios días sin actualizar, CVEs publicados recientemente no aparecerán en los resultados.

> La primera ejecución de `gvm-setup` descarga cientos de miles de NVTs y datos SCAP. Es normal que tarde 20-45 minutos y que el proceso consuma bastante CPU y disco durante ese tiempo.
