# Suite de hxctools

> hxctools es una suite de herramientas para capturar convertir y manipular el tráfico Wi-Fi, especialmente para handshakes WPA/WPA2/WPA3, PMKID y ataques offline.

## Para que se usa hxctools

* Procesar capturas Wi-Fi
* Extraer handshakes o PMKID
* Convertir las capturas a un formato compatible en hashcat
* Manipular hashes

## Diferencias de hxctools y aircrack-ng

| Enfoque moderno         | Enfoque clásico            |
| ----------------------- | -------------------------- |
| PMKID y WPA3            | WPA/WPA2 principalmente    |
| Flujo modular           | Suite monolítica           |
| Optimizado para hashcat | Propio sistema de cracking |

## Herramientas principales de la suite

| hcxdumptool   | Captura activa o pasiva de tráfico Wi-Fi |
| ------------- | ---------------------------------------- |
| hcxpcapngtool | Procesar capturas y extraer hashes       |
| hcxhashtool   | Gestionar y convertir hashes             |
| hcxhash2cap   | Convertir hashes a capturas              |
| hcxwltool     | Manipular wordlists                      |
| hcxmactool    | Gestionar y analizar direcciones MAC     |

### hcxdumptool

```
hcxdumptool -i wlan0mon -o captura.pcapng
```

#### Funciones principales

* Capturar tráfico Wi-Fi
* obtener handshakes
* provocar tráfico&#x20;

| Interfaz Wi-Fi        | `-i <iface>`        | Interfaz en modo monitor                                |
| --------------------- | ------------------- | ------------------------------------------------------- |
| Archivo de salida     | `-o <file>`         | Captura en formato `.pcapng`                            |
| Captura PMKID         | `--enable_status=1` | Habilita estado y PMKID                                 |
| Escaneo activo        | `--active_beacon`   | Envío de beacons activos                                |
| Desautenticación      | `--deauth <num>`    | desauntentica dispositivos para forzar las reconexiones |
| Filtro por canal      | `-c <list>`         | Canales a monitorizar                                   |
| Filtro BSSID          | `--filterlist_ap`   | Filtra puntos de acceso                                 |
| Captura pasiva        | `--passive`         | Solo escucha tráfico                                    |
| Intervalo de rotación | `--rotate <sec>`    | Rota archivos de captura                                |
| Guardar clientes      | `--client`          | Registra estaciones (STA)                               |

### hcxpcapngtool

```
hcxpcapngtool -o hashes.hc22000 captura.pcapng
```

#### Funciones principales

* Convertir archivos <kbd>.pcap</kbd> o <kbd>.pcapng</kbd> a formato compatible con hashcat
* Extraer handshakes WPA/WPA2
* Extraer PMKID
* Filtrar redes y estaciones
* Generar hashes para ataques offline

| Entrada de captura   | `-i <file>`      | Archivo `.pcap` / `.pcapng` de entrada |
| -------------------- | ---------------- | -------------------------------------- |
| Salida hashcat       | `-o <file>`      | Genera hashes en formato `.hc22000`    |
| Filtrar ESSID        | `--essid <name>` | Procesa solo una red concreta          |
| Filtrar BSSID        | `--bssid <mac>`  | Filtra por punto de acceso             |
| Eliminar duplicados  | `--all`          | Incluye todos los handshakes válidos   |
| Mostrar estadísticas | `--info`         | Información de redes y clientes        |



### hcxhashtool

```
hcxhashtool -i hashes.hc22000 -o hashes_limpios.hc22000
```

#### Funciones principales

* Filtrar hashes duplicados
* Convertir formatos
* Limpiar hashes inválidos

| Entrada             | `-i <file>`           | Archivo de hashes de entrada |
| ------------------- | --------------------- | ---------------------------- |
| Salida              | `-o <file>`           | Archivo de hashes limpio     |
| Eliminar duplicados | `--remove-duplicates` | Quita hashes repetidos       |
| Mostrar info        | `--info`              | Estadísticas del archivo     |
| Convertir formato   | `--convert`           | Conversión entre formatos    |

### hcxhash2cap

```
hcxhash2cap --pmkid hashes.hc22000 -o salida.pcapng
```

#### Funciones principales

* Convertir hashes a archivos <kbd>.pcap</kbd> o <kbd>.pcapng</kbd>&#x20;
* Analizar la captura con wireshark

| Entrada PMKID     | `--pmkid <file>`  | Archivo de hashes PMKID    |
| ----------------- | ----------------- | -------------------------- |
| Entrada handshake | `--hccapx <file>` | Hashes de handshakes       |
| Salida            | `-o <file>`       | Archivo `.pcapng` generado |

### hcxwltool

```
hcxwltool -i wordlist.txt -o limpia.txt
```

#### Funciones principales

* Trabajar con Wordlists
* Eliminar duplicados
* Filtrar por longitud

| Entrada             | `-i <file>`      | Wordlist original           |
| ------------------- | ---------------- | --------------------------- |
| Salida              | `-o <file>`      | Wordlist procesada          |
| Eliminar duplicados | `--dedup`        | Quita palabras repetidas    |
| Filtrar longitud    | `--min`, `--max` | Longitud mínima y máxima    |
| Normalizar          | `--clean`        | Limpia caracteres inválidos |

### hcxmactool

```
hcxmactool -i macs.txt
```

#### Funciones principales

* Identificar fabricante
* Filtrar por OUI
* Analizar patrones de MAC aleatoria

| Entrada             | `-i <file>`      | Wordlist original           |
| ------------------- | ---------------- | --------------------------- |
| Salida              | `-o <file>`      | Wordlist procesada          |
| Eliminar duplicados | `--dedup`        | Quita palabras repetidas    |
| Filtrar longitud    | `--min`, `--max` | Longitud mínima y máxima    |
| Normalizar          | `--clean`        | Limpia caracteres inválidos |

