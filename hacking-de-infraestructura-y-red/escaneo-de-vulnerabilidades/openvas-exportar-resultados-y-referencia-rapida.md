# OpenVAS — Exportar Resultados y Referencia Rápida

### Exportar resultados

GVM permite exportar los resultados de un escaneo en varios formatos para incluir en informes o procesarlos externamente.

#### Desde la interfaz web (GSA)

Scans → Reports → seleccionar el informe → botón de descarga (icono con flecha)

Seleccionar el formato en el desplegable:

| Formato          | Uso                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------- |
| **PDF**          | Informe visual para clientes y dirección. Incluye gráficas y tablas de hallazgos.                        |
| **XML**          | Formato nativo de GVM. Contiene todos los datos del escaneo, reutilizable en GVM.                        |
| **CSV**          | Tabla plana de todos los hallazgos. Para análisis en Excel, Python o integración con otras herramientas. |
| **HTML**         | Informe web navegable completo.                                                                          |
| **ITG**          | Formato de intercambio para sistemas de tickets.                                                         |
| **TXT**          | Texto plano básico.                                                                                      |
| **Verinice XML** | Para importar en Verinice (herramienta de gestión de riesgos).                                           |
| **Topology SVG** | Mapa de red en formato SVG con los hosts descubiertos.                                                   |

#### Configurar el informe antes de exportar

Antes de exportar conviene aplicar filtros para que el informe contenga solo los hallazgos relevantes:

```
Filtros recomendados para un informe ejecutivo:
→ Severity: High y Critical únicamente
→ QoD: ≥ 70%

Filtros para informe técnico completo:
→ Severity: Medium en adelante
→ QoD: ≥ 50%
→ Ordenar por: Severity (descendente)
```

#### Exportar desde línea de comandos con gvm-cli

Para automatización o integración con pipelines de seguridad:

```bash
# Instalar gvm-cli
pip3 install gvm-tools

# Listar todos los reports disponibles
gvm-cli socket --gmp-username admin --gmp-password contraseña \
    --xml "<get_reports/>"

# Exportar un report específico en CSV
gvm-cli socket --gmp-username admin --gmp-password contraseña \
    --xml "<get_reports report_id='REPORT-UUID' format_id='c1645568-627a-11e3-a660-406186ea4fc5'/>" \
    > resultados.csv

# Exportar en PDF
gvm-cli socket --gmp-username admin --gmp-password contraseña \
    --xml "<get_reports report_id='REPORT-UUID' format_id='c402cc3e-b531-11e1-9163-406186ea4fc5'/>" \
    > informe.pdf

# IDs de formato más usados:
# CSV:  c1645568-627a-11e3-a660-406186ea4fc5
# PDF:  c402cc3e-b531-11e1-9163-406186ea4fc5
# XML:  a994b278-1f62-11e1-96ac-406186ea4fc5
```

#### Procesar el CSV con Python

```python
import pandas as pd

df = pd.read_csv('resultados.csv')

# Ver las columnas disponibles
print(df.columns.tolist())

# Filtrar solo Critical y High
criticos = df[df['Severity'].isin(['High', 'Critical']) | (df['CVSS'].astype(float) >= 7.0)]

# Agrupar hallazgos por host
por_host = df.groupby('IP')['Severity'].value_counts().unstack(fill_value=0)
print(por_host)

# Top 10 vulnerabilidades más críticas
top10 = df.sort_values('CVSS', ascending=False).head(10)[['IP', 'NVT Name', 'CVSS', 'CVEs']]

# Exportar solo CVEs únicos afectados
df[['CVEs', 'NVT Name', 'Severity']].drop_duplicates().to_csv('cves_unicos.csv', index=False)
```

### Gestión de falsos positivos — Override

GVM tiene un sistema de **Overrides** para marcar hallazgos como falsos positivos o cambiar su severidad, similar a las Plugin Rules de Nessus.

Scans → Reports → hallazgo → icono de override (lapiz con símbolo)

```
Opciones de override:
→ Cambiar la severidad a cualquier nivel (incluyendo Log para "ignorar")
→ Aplicar el override solo a un host específico o a todos
→ Aplicar solo a un puerto/protocolo específico
→ Añadir un comentario justificando el cambio
→ Definir una fecha de expiración del override
```

Los overrides se aplican a todos los reportes futuros automáticamente. Muy útil cuando se confirma que un hallazgo es un falso positivo recurrente.

### Tickets de remediación

GVM tiene un sistema de tickets integrado para el seguimiento de la remediación:

Resilience → Remediation Tickets → New Ticket desde un hallazgo

```
→ Asignar el ticket a un usuario responsable de remediarlo
→ Añadir nota con el plan de acción
→ Estado: Open → In Progress → Closed
→ Fecha límite de resolución
```

Útil en entornos donde se quiere llevar el ciclo completo de detección → asignación → verificación de remediación dentro de la misma herramienta.

### Comparar escaneos

GVM muestra automáticamente la evolución entre escaneos del mismo target. En la vista de Reports:

```
→ Delta Report: comparar dos reportes específicos
→ Muestra: vulnerabilidades nuevas, resueltas y persistentes
→ Permite ver si la remediación fue efectiva entre auditorías
```

### Referencia rápida — Flujo completo

```
1. Iniciar los servicios
   sudo gvm-start
   → Acceder a https://127.0.0.1:9392

2. Crear credenciales (si el escaneo es autenticado)
   Configuration → Credentials → New Credential

3. Crear el Target
   Configuration → Targets → New Target
   → Hosts: IP o rango CIDR
   → Asignar credenciales creadas

4. Crear la Task
   Scans → Tasks → New Task
   → Target: seleccionar el creado
   → Scan Config: Full and Fast (uso general)

5. Lanzar la Task
   → Botón play en la Task
   → Monitorizar progreso en tiempo real

6. Analizar resultados
   Scans → Reports → informe más reciente
   → Filtrar por severidad ≥ High y QoD ≥ 70%
   → Revisar detalle de cada hallazgo crítico

7. Exportar
   → PDF para el informe
   → CSV para análisis
   → XML para reimportar o archivar
```

### Preguntas tipo de la evaluación

Las evaluaciones de HTB sobre OpenVAS suelen pedir:

* Número de hosts activos encontrados en el escaneo
* Número total de vulnerabilidades por nivel de severidad
* Nombre o CVE de una vulnerabilidad específica encontrada
* QoD de un hallazgo concreto
* Puerto en que se detectó un servicio vulnerable
* Solución recomendada para un hallazgo específico
* Versión de software vulnerable detectada en el output

Para responder estas preguntas usar la combinación de:

1. **Dashboard del Report** → totales y gráficas de distribución
2. **Pestaña Results** → lista completa filtrable
3. **Detalle del hallazgo** → CVE, CVSS, output con evidencia, solución

### Comandos de gestión de servicios GVM

```bash
# Iniciar todos los servicios
sudo gvm-start

# Detener todos los servicios
sudo gvm-stop

# Verificar estado
sudo gvm-check-setup

# Reiniciar si hay problemas
sudo gvm-stop && sudo gvm-start

# Ver logs si algo falla
sudo journalctl -u ospd-openvas -f
sudo journalctl -u gvmd -f
sudo journalctl -u gsad -f

# Cambiar contraseña del admin
sudo gvmd --user=admin --new-password=nuevacontraseña

# Crear usuario adicional
sudo gvmd --create-user=usuario2 --password=contraseña

# Forzar actualización de feeds
sudo greenbone-nvt-sync
sudo greenbone-scapdata-sync
sudo greenbone-certdata-sync
```

> Si la interfaz web de GSA tarda en cargar o no responde, revisar que todos los servicios están activos con `sudo gvm-check-setup`. Es habitual que tras reiniciar el sistema los servicios no arranquen solos — hay que lanzar `sudo gvm-start` manualmente.

> GVM almacena los resultados de todos los escaneos en PostgreSQL. Si el disco se llena (los feeds y resultados ocupan bastante), los escaneos pueden fallar silenciosamente. Revisar el espacio disponible periódicamente con `df -h`.
