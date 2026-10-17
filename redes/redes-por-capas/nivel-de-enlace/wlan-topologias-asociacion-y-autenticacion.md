# WLAN: topologías, asociación y autenticación

### Modos de funcionamiento

<table><thead><tr><th width="193.41796875">Modo</th><th>Descripción</th></tr></thead><tbody><tr><td>Ad hoc</td><td>Los equipos se hablan directamente, sin AP</td></tr><tr><td>Infraestructura</td><td>Los equipos se conectan a un AP que los integra en la red</td></tr></tbody></table>

### Conjuntos de servicio

#### BSS (Basic Service Set)

Un AP y sus clientes asociados se identifica con un BSSID (MAC de 48 bits del AP), se usa en redes domésticas.

<figure><img src="../../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

#### IBSS (Independent BSS)

Red ad hoc sin AP.

<figure><img src="../../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

#### ESS (Extended Service Set)

Varios BSS interconectados con la misma red, se usa en empresas, hospitales, centros educativos.

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

Roaming: el cliente cambia de AP dentro de un ESS manteniendo la conexión, siempre que la infraestructura esté bien configurada.

SSID es el nombre visible de la red; BSSID identifica a cada AP concreto.

### Cómo se conecta un cliente

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

### Seguridad

<table><thead><tr><th width="260.3359375">Mecanismo</th><th>Nivel de seguridad</th></tr></thead><tbody><tr><td>WEP</td><td>Obsoleto</td></tr><tr><td>WPA2</td><td>Válido</td></tr><tr><td>WPA3</td><td>Recomendado, más resistente frente a ataques sobre contraseñas</td></tr></tbody></table>
