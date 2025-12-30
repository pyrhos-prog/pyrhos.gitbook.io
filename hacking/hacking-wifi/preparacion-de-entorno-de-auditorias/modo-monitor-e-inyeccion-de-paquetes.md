# Modo monitor e inyección de paquetes

## Modo monitor&#x20;

El modo monitor o modo promiscuo es un modo de funcionamiento de una antena wifi que permite escuchar todos los paquetes que hay en el aire, no solo los que nos envia nuestros router sino también el intercambio de información que hay en otras redes Wi-Fi al alcance.

#### Funciones

* Capturar paquetes
* Ver las direcciones MAC de los dispositivos alrededor
* Catpurar las tramas Wi-Fi

Dependiendo del chipset de tu tarjeta Wi-Fi podrás o no usar el modo monitor



**Estos son los chipset y modelos mas usados para auditorias Wi-Fi**

| **Chipset**            | **Modelos Recomendados**                             | **Nivel de Compatibilidad**                          |
| ---------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Atheros AR9271**     | <p>• Alfa AWUS036NHA<br>• Panda PAU09</p>            | **Excelente** - Soporte nativo en Kali Linux         |
| **Realtek RTL8812AU**  | <p>• Alfa AWUS036ACH<br>• TP-Link AC1200 T3U</p>     | **Excelente** - Compatible con drivers actualizados  |
| **Realtek RTL8821AU**  | <p>• Alfa AWUS036ACS<br>• Netgear A6150</p>          | **Excelente** - Amplio soporte en distribuciones     |
| **Realtek RTL8814AU**  | <p>• Alfa AWUS1900<br>• EDUP AC1900</p>              | **Excelente** - Ideal para auditorías avanzadas      |
| **Ralink RT3070**      | <p>• Alfa AWUS036NH<br>• Panda PAU06</p>             | **Buena** - Soporte estable pero tecnología anterior |
| **Realtek RTL8188EUS** | <p>• Alfa AWUS036EAC<br>• TP-Link AC600 T2U Plus</p> | **Buena** - Requiere drivers adicionales             |
| **Atheros AR9462**     | <p>• Qualcomm QCA9377<br>• Intel 7265 (limitado)</p> | **Variable** - Depende de la implementación          |

### Activar el modo monitor

Para activar el modo monitor en la tarjeta hay que tener la suite de aircrack-ng instalada en el sistema, una vez la tengamos hay que saber cual es el nombre de nuestra antena.

```
ifconfig
<terminar>
```

El siguiente paso es utilizar airmon-ng para activar el modo monitor en la antena.

```
sudo airmon-ng start wlan0 # El nombre de tu antena puede ser direferente a wlan0
<terminar>
```

Con esto ya esta activado el modo monitor para poder a escanear las redes Wi-Fi de alrededor.

