# Nessus — Introducción y Primeros Pasos

Nessus es el escáner de vulnerabilidades más utilizado en el mundo, desarrollado por **Tenable**. Automatiza la detección de vulnerabilidades conocidas, misconfiguraciones, software desactualizado y problemas de cumplimiento normativo en hosts, redes y aplicaciones.

### ¿Qué es el escaneo de vulnerabilidades?

El escaneo de vulnerabilidades es el proceso de identificar de forma automatizada debilidades de seguridad en sistemas y redes. A diferencia del pentesting manual, un escáner envía sondas a los sistemas objetivo y compara las respuestas contra una base de datos de vulnerabilidades conocidas (CVEs) para determinar qué versiones de software están presentes y si son vulnerables.

El resultado es una lista priorizada de hallazgos con niveles de severidad, descripción del problema y recomendaciones de remediación.

#### Diferencia entre escaneo de vulnerabilidades y pentesting

|                    | Escaneo de vulnerabilidades       | Pentesting                                  |
| ------------------ | --------------------------------- | ------------------------------------------- |
| **Objetivo**       | Identificar debilidades conocidas | Explotar debilidades para demostrar impacto |
| **Profundidad**    | Amplio, superficie completa       | Profundo, rutas específicas                 |
| **Automatización** | Alta                              | Baja — requiere juicio humano               |
| **Resultado**      | Lista de CVEs con severidad       | Cadena de ataque demostrada                 |
| **Frecuencia**     | Continua o periódica              | Puntual (por proyecto)                      |

### Ediciones de Nessus

| Edición                 | Uso                              | Límite de hosts | Precio     |
| ----------------------- | -------------------------------- | --------------- | ---------- |
| **Nessus Essentials**   | Aprendizaje y uso personal       | 16 IPs          | Gratuita   |
| **Nessus Professional** | Pentesting y auditorías          | Sin límite      | De pago    |
| **Nessus Expert**       | Cloud, IaC, superficies externas | Sin límite      | De pago    |
| **Tenable.io**          | Plataforma cloud empresarial     | Sin límite      | SaaS       |
| **Tenable.sc**          | Gestión empresarial on-premise   | Sin límite      | Enterprise |

Para aprendizaje y laboratorios, **Nessus Essentials** es suficiente.

### Instalación

```bash
# Descargar desde https://www.tenable.com/downloads/nessus
# Seleccionar el paquete para la distribución correspondiente

# Debian / Ubuntu / Kali
dpkg -i Nessus-*.deb
systemctl start nessusd
systemctl enable nessusd

# RHEL / Fedora / CentOS
rpm -ivh Nessus-*.rpm
systemctl start nessusd
systemctl enable nessusd

# Verificar que el servicio está corriendo
systemctl status nessusd
```

Una vez iniciado el servicio, acceder a la interfaz web en `https://localhost:8834`. El navegador mostrará una advertencia de certificado autofirmado — es normal, continuar de todas formas.

### Configuración inicial

Al acceder por primera vez, Nessus guía a través de un asistente de configuración:

1. Seleccionar la edición (Nessus Essentials para uso personal)
2. Crear una cuenta de administrador (usuario y contraseña)
3. Introducir el código de activación (se obtiene registrándose en tenable.com)
4. Esperar a que Nessus descargue e instale los plugins — este proceso tarda **varios minutos** en la primera instalación

Los **plugins** son los scripts individuales que Nessus ejecuta para detectar cada vulnerabilidad. En 2024 Nessus tiene más de 180.000 plugins activos que se actualizan diariamente.

### Interfaz principal

Una vez configurado, la interfaz tiene estas secciones principales:

| Sección          | Descripción                                               |
| ---------------- | --------------------------------------------------------- |
| **My Scans**     | Lista de escaneos creados y sus resultados                |
| **All Scans**    | Vista global de todos los escaneos                        |
| **Policies**     | Plantillas personalizadas reutilizables                   |
| **Plugin Rules** | Reglas para modificar la severidad de plugins específicos |
| **Scanners**     | Gestión de escáneres remotos (Nessus Pro/Expert)          |
| **Settings**     | Configuración general, actualizaciones, SMTP para alertas |

> Nessus requiere que los plugins estén actualizados para detectar vulnerabilidades recientes. Actualizar manualmente desde Settings → Software Update si el sistema no tiene acceso automático a internet.

> El escaneo de vulnerabilidades solo debe realizarse sobre sistemas propios o con autorización explícita por escrito. Escanear sistemas sin permiso es ilegal en la mayoría de jurisdicciones.
