# Cisco Packet Tracer

Cisco Packet tracer es un simulador de redes con dispositivos cisco enfocado en el aprendizaje de la configuración de redes.

## Cisco IOS Router - CLI

### MODOS DE ACCESO

| Comando                               | Descripción                           |
| ------------------------------------- | ------------------------------------- |
| `Router>enable`                       | Entrar a modo privilegiado            |
| `Router#config terminal`              | Entrar a modo de configuración global |
| `Router#show run`                     | Ver configuración activa              |
| `Router#copy run start` / `Router#wr` | Guardar cambios                       |

### CONFIGURACIÓN GENERAL

#### Nombre de host

```
Router(config)#hostname myRouter
```

#### Contraseñas

```
# Consola
Router(config)#line console 0
Router(config-line)#password cisco
Router(config-line)#login

# Modo privilegiado
Router(config)#enable password cisco
Router(config)#enable secret cisco       ← cifrada (preferida)

# Telnet (VTY)
Router(config)#line vty 0 4
Router(config-line)#password cisco
Router(config-line)#login
```

#### SSH

```
Router(config)#hostname R2
Router(config)#ip domain-name cisco.com
Router(config)#crypto key generate rsa
Router(config)#username felipe secret cisco
Router(config)#line vty 0 4
Router(config-line)#transport input ssh
Router(config-line)#login local
Router(config)#ip ssh time-out 15
Router(config)#ip authentication-retries 2
```

### INTERFACES

#### FastEthernet

```
Router(config)#interface fastEthernet 0/0
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown
```

#### Serial DCE (proporciona reloj)

```
Router(config)#interface Serial 0/0/0
Router(config-if)#ip address 192.168.50.5 255.255.255.0
Router(config-if)#clockrate 56000
Router(config-if)#no shutdown
```

#### Serial DTE

```
Router(config)#interface Serial 0/0/0
Router(config-if)#ip address 192.168.50.7 255.255.255.0
Router(config-if)#no shutdown
```

#### Loopback

```
Router(config)#interface loopback 0
Router(config-if)#ip address <ip> <mask>
```

> **Verificar DCE/DTE:** `Router#show controllers serial 0/0/0`

### DHCP

#### Configurar servidor DHCP

```
Router(config)#ip dhcp pool POOL1
Router(dhcp-config)#network 192.168.1.0 255.255.255.0
Router(dhcp-config)#dns-server 192.168.1.10
Router(dhcp-config)#default-router 192.168.1.1
Router(dhcp-config)#exit
Router(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.49
```

#### Interfaz como cliente DHCP

```
Router(config)#interface f0/0
Router(config-if)#ip address dhcp
Router(config-if)#no shutdown
```

#### DHCP Relay

```
Router(config)#interface f0/0
Router(config-if)#ip helper-address 192.168.2.1
```

#### Verificación

```
Router#show ip dhcp binding
Router#show ip dhcp pool
Router#debug ip dhcp server events
```

### ENRUTAMIENTO

#### Rutas estáticas

```
Router(config)#ip route <red_destino> <mask> <next-hop | interfaz_salida>

# Ruta por defecto (quad-zero)
Router(config)#ip route 0.0.0.0 0.0.0.0 <next-hop>
```

#### RIP

```
Router(config)#router rip
Router(config-router)#version 2
Router(config-router)#network <red_directamente_conectada>
Router(config-router)#no auto-summary
Router(config-router)#passive-interface FastEthernet 0/0
Router(config-router)#default-information originate   ← propaga ruta default
Router(config-router)#redistribute static
```

#### EIGRP

```
Router(config)#router eigrp <AS>
Router(config-router)#network <red> [wildcard-mask]

# Modificar ancho de banda
Router(config-if)#bandwidth <kilobits>
```

#### OSPF

```
Router(config)#router ospf <process-id>
Router(config-router)#network <red> <wildcard> area <area-id>
Router(config-router)#router-id <ip>
Router(config-router)#default-information originate

# Modificar costo
Router(config-if)#ip ospf cost 1562

# Prioridad DR/BDR
Router(config-if)#ip ospf priority <0-255>

# Intervalos
Router(config-if)#ip ospf hello-interval <segundos>
Router(config-if)#ip ospf dead-interval <segundos>

# Reiniciar proceso
Router#clear ip ospf process
```

#### Verificación de rutas

```
Router#show ip route
Router#show ip protocols
Router#show ip interface brief
Router#debug ip routing
Router#ping <destino>
```

### ACL — Listas de Control de Acceso

#### ACL Estándar (numerada)

```
Router(config)#access-list 10 permit 192.168.10.0 0.0.0.255
Router(config)#no access-list 10

# Aplicar a interfaz
Router(config-if)#ip access-group 10 {in | out}

# Restringir acceso VTY
Router(config-line)#access-class 10 in
```

#### ACL Estándar (nombrada)

```
Router(config)#ip access-list standard NO_ACCESS
Router(config-std-nacl)#deny host 192.168.11.10
Router(config-std-nacl)#permit 192.168.10.0 0.0.0.255
Router(config-if)#ip access-group NO_ACCESS out
```

#### ACL Extendida

```
Router(config)#access-list 103 permit tcp 192.168.10.0 0.0.0.255 any eq 80
Router(config)#access-list 103 permit tcp 192.168.10.0 0.0.0.255 any eq 443
Router(config)#access-list 104 permit tcp any 192.168.10.0 0.0.0.255 established
```

#### ACL Extendida (nombrada)

```
Router(config)#ip access-list extended SURFING
Router(config-ext-nacl)#permit tcp 192.168.10.0 0.0.0.255 any eq 80
Router(config-ext-nacl)#permit tcp 192.168.10.0 0.0.0.255 any eq 443
```

#### ACL Basada en tiempo

```
Router(config)#time-range THREEDAYS
Router(config-time-range)#periodic Monday Wednesday Friday 8:00 to 17:00
Router(config)#access-list 101 permit tcp 192.168.10.0 0.0.0.255 any eq telnet time-range THREEDAYS
```

#### ACL Reflexiva

```
Router(config)#ip access-list extended OUTBOUNDFILTERS
Router(config-ext-nacl)#permit tcp 192.168.0.0 0.0.255.255 any reflect TCPTRAFFIC
Router(config)#ip access-list extended INBOUNDFILTERS
Router(config-ext-nacl)#evaluate TCPTRAFFIC
Router(config-if)#ip access-group INBOUNDFILTERS in
Router(config-if)#ip access-group OUTBOUNDFILTERS out
```

### PPP (WAN)

```
Router(config-if)#encapsulation ppp
Router(config-if)#compress [predictor | stac]
Router(config-if)#ppp quality <porcentaje>
Router(config-if)#ppp authentication {chap | pap | chap pap}
Router(config-if)#ppp multilink

Router#show interfaces serial
Router#debug ppp
Router#undebug all
```

### FRAME RELAY

```
Router(config-if)#encapsulation frame-relay
Router(config-if)#bandwidth 64
Router(config-if)#no frame-relay inverse-arp
Router(config-if)#frame-relay map ip <ip_destino> <dlci> broadcast
Router(config-if)#frame-relay lmi-type [cisco | ansi | q933a]

# Subinterfaz punto a punto
Router(config)#interface serial 0/0/0.103 point-to-point
Router(config-subif)#frame-relay interface-dlci 103

# Verificación
Router#show frame-relay map
Router#show frame-relay lmi
Router#show frame-relay pvc 102
```

### RECUPERACIÓN DE CONTRASEÑA (Router)

```
1. Detener boot → Ctrl+Break
2. rommon> confreg 0x2142
3. rommon> reset
4. Router#enable
5. Router#erase startup-config
6. Router(config)#config-register 0x2102
7. Router#reload
```

### COMANDOS SHOW

| Comando                         | Descripción                        |
| ------------------------------- | ---------------------------------- |
| `show run`                      | Configuración activa               |
| `show ip route`                 | Tabla de rutas                     |
| `show ip interface brief`       | Estado resumido de interfaces      |
| `show ip protocols`             | Protocolos de enrutamiento activos |
| `show ip ospf neighbor`         | Vecinos OSPF                       |
| `show ip eigrp neighbors`       | Vecinos EIGRP                      |
| `show ip eigrp topology`        | Topología EIGRP                    |
| `show ip rip database`          | Base de datos RIP                  |
| `show ip dhcp binding`          | Asignaciones DHCP                  |
| `show interfaces serial 0/0/0`  | Estado de interfaz serial          |
| `show controllers serial 0/0/0` | Info DCE/DTE                       |
| `show frame-relay map`          | Mapa Frame Relay                   |
