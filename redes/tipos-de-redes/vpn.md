---
icon: chart-network
---

# VPN

Una VPN a nivel de red crea un túnel cifrado entre redes o dispositivos a nivel de capa 3 (Red) del modelo OSI, permitiendo la transmisión segura de tráfico IP sobre redes no confiables.

* Encapsula paquetes IP
* Cifra el tráfico
* Garantiza confidencialidad, integridad y autenticación
* Opera antes de que las aplicaciones procesen los datos

Todo el tráfico de red se envía a través del túnel VPN de forma transparente para las aplicaciones.

### Capas del modelo OSI implicadas

* **Capa 3 (Red)**: IP
* **Capa 4 (Transporte)**: UDP / TCP (dependiendo del protocolo)
* **Capa 7 (Aplicación)**: solo para control o autenticación

### Tipos de VPN

#### VPN Site-to-Site

Conecta dos redes completas entre sí.

**Ejemplo**:

* Sede central ↔ Sucursal

**Características**:

* Transparente para los usuarios
* Normalmente permanente
* Se configura en routers o firewalls

#### VPN Remote Access

Permite a un cliente remoto acceder a una red interna.

**Ejemplo**:

* Usuario remoto ↔ Red corporativa

**Características**:

* Requiere cliente VPN
* Autenticación de usuario
* Acceso controlado por políticas

### Protocolos VPN

{% tabs %}
{% tab title="IPsec" %}
Protocolo estándar para VPN

**Componentes**:

* AH (Authentication Header)
* ESP (Encapsulating Security Payload)
* IKE (Internet Key Exchange)

**Modos**:

* Modo túnel
* Modo transporte

**Ventajas**:

* Muy seguro
* Estándar industrial

**Desventajas**:

* Configuración compleja
{% endtab %}

{% tab title="OpenVPN" %}


VPN basada en SSL/TLS.

**Características**:

* Usa TLS para cifrado
* Opera sobre UDP o TCP
* Flexible y ampliamente usada

**Ventajas**:

* Alta compatibilidad
* Fácil de auditar
{% endtab %}

{% tab title="Wireguard" %}
VPN moderna y ligera.

**Características**:

* Criptografía moderna
* Código reducido
* Muy alto rendimiento

**Ventajas**:

* Configuración sencilla
* Menor superficie de ataque
{% endtab %}
{% endtabs %}

### Métodos de autenticación

* Certificados digitales
* Usuario y contraseña
* Claves precompartidas (PSK)
* Autenticación multifactor (MFA)

### Ventajas de las VPN

* Protección de tráfico en redes inseguras
* Acceso remoto seguro
* Transparencia para aplicaciones
* Segmentación de redes

### Desventajas

* Overhead de cifrado
* Dependencia de la configuración
* Posible cuello de botella
* Riesgo si las credenciales se comprometen

### Uso en entornos corporativos

* Teletrabajo
* Interconexión de sedes
* Acceso a recursos internos
* Protección del tráfico interno
