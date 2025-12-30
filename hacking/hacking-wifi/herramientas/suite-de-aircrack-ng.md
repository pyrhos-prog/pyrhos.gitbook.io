# Suite de aircrack-ng

> La suite de aircrack-ng son un conjunto de herramientas de hacking de redes Wi-Fi rápidas y eficientes, con muchas funcionalidades.

## Usos principales

* **Captura e inyección de paquetes**: permite capturar paquetes de las redes Wi-Fi cercanas y la inyección de paquetes como los de desautenticación.
* **Cracking WEP y WPA WPA2-PSK:** permite descrifrar claves WEP y WPA2-PSK mediante el uso de paquetes capturados y ataques de fuerza bruta para descifrar la contraseña.
* **Monitoreo de red**: se puede monitorear la red en tiempo real para identificar dispositivos conectados, direcciones MAC y exportar esos datos.

## Uso de la suite de aircrack-ng

### 1. Capturar paquetes&#x20;

Con este comando podemos capturar paquetes para guardarlos con el fin de cracking de la contraseña o analisis de dispositivos y la red.

```
airodump-ng <interfaz>
```

### 2. Crackear clave WEP

Con este comando podemos descifrar una contraseña con cifrado WEP utilizando los paquetes capturados.

```
aircrack-ng -b <BSSID> <paquetes_capturados>
```

### 3. Crackear clave WPA/WPA2-PSK con diccionario

Se puede descifrar una contrasña con cifrado WPA/WPA2-PSK  utilizando un diccionario, los paquetes capturados y el BSSID del AP.

```
aircrack-ng -w <diccionario> -b <BSSID> <paquetes_capturados>
```

## 4. Deauth de clientes

Para desauntenticar a los clientes de una red Wi-Fi podemos utilizar paquetes de desautenticación y romper el handshake, asi tendrán que reconectarse al router y nos permitirá capturar el handshake para hackear la red Wi-Fi.

```
aireplay-ng --deauth <nº paquetes> -a <BSSID> -c <MAC> <interfaz>
```
