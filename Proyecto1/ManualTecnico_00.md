# **Manual Tecnico**
## Grupo 18
| Carnet | Nombre |
| -- | -- |
| 201830313 | Denilson Florentín de León Aguilar |
| 201700377 | Erick Omar Letona Figueroa | 
---
# **Topología**
# VLAN's
| VLAN | Nombre | Red | Submask |
| -- | -- | -- | -- |
| 10 | naranja18 |192.168.10.0 | 255.255.255.255 |
| 20 | verde18 | 192.168.20.0 | 255.255.255.255 |

# DHCP
**Server-PT DHCP1**
| IP | Configuration |
| -- | -- |
| **Ip Address:** | 192.168.10.4 | 
| **Subnet Mask:** | 255.255.255.0 |

| DHCP |  Services |
| -- | -- |
| **Pool Name:** | serverpool |
| **Ip Address:** | 192.168.10.1 | 
| **Start Ip Address:** | 192.168.10.10 |
| **Maximum Number of Users:** | 248 |

**Server-PT DHCP2**
| IP | Configuration |
| -- | -- |
| **Ip Address:** | 192.168.20.4 | 
| **Subnet Mask:** | 255.255.255.0 |

| DHCP |  Services |
| -- | -- |
| **Pool Name:** | serverpool |
| **Ip Address:** | 192.168.20.1 | 
| **Start Ip Address:** | 192.168.20.10 |
| **Maximum Number of Users:** | 248 |
---
# **Comandos de configuración**
## EIGRP configuration
**MSW Edificio Principal**
```js
conf t
// VTP
vtp domain g18
vtp password admin
vtp mode server

// vlan
vlan 10
name naranja18
vlan 20
name verde18
exit
interface vlan 10
ip address 192.168.10.5 255.255.255.0
no shutdown
interface vlan 20
ip address 192.168.20.5 255.255.255.0
no shutdown
exit
interface range GigabitEthernet1/0/3-4
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
description ACC_VLANs
no shutdown
exit
ip routing
router eigrp 100
network 192.168.10.0 0.0.0.255 
network 192.168.20.0 0.0.0.255 
no auto-summary
exit
end
```

**MSW Edificio 1**
```js
conf t
// VTP
vtp domain g18
vtp password admin
vtp mode client

// vlan
vlan 10
name naranja18
vlan 20
name verde18
exit

// VLANS
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown
exit

// Interfaces
interface range GigabitEthernet1/0/1-3
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
description ACC_VLANs
no shutdown
exit

// LACP
interface range gigabitEthernet1/0/4-6
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
channel-group 3 mode active
end

// EIGRP
ip routing
router eigrp 100
network 192.168.10.0 0.0.0.255 
network 192.168.20.0 0.0.0.255 
no auto-summary
exit
end
```

**MSW Edificio 2**
```js
conft t
// VTP
vtp domain g18
vtp password admin
vtp mode client

// vlan
vlan 10
name naranja18
exit
vlan 20
name verde18
exit
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown
exit

// Interfaces
interface range GigabitEthernet1/0/1-3
switchport mode trunk
switchport trunk allowed vlan all
description ACC_VLANs
no shutdown
exit

// LACP
interface range gigabitEthernet1/0/4-6
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
channel-group 3 mode active

// EIGRP
ip routing
router eigrp 100
network 192.168.10.0 0.0.0.255 
network 192.168.20.0 0.0.0.255 
no auto-summary
exit
end
```

**MSW Edificio 3**
```js
conft t

// VTP
vtp domain g18
vtp password admin
vtp mode client

// vlan
vlan 10
name naranja18
exit
vlan 20
name verde18
exit
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown
exit
interface range GigabitEthernet1/0/2-3
switchport mode trunk
switchport trunk allowed vlan all
description ACC_VLANs
no shutdown
exit
ip routing
router eigrp 100
network 192.168.10.0 0.0.0.255 
network 192.168.20.0 0.0.0.255 
no auto-summary
exit
end
```

**Edificio 1**
```js
conf t

// VTP
vtp domain g18
vtp password admin
vtp mode client

// vlan
vlan 10
name naranja18
vlan 20
name verde18
exit
interface range gigabitEthernet1/0/4-6
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
channel-group 3 mode active
end
```

**Edificio 2**
```js
conf t
// VTP
vtp domain g18
vtp password admin
vtp mode client

// vlan
vlan 10
name naranja18
vlan 20
name verde18
exit
interface range gigabitEthernet1/0/4-6
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
channel-group 3 mode active
end
```
### Redes de Cada Edificio (MSW)
#### **MSW 3**
**Interfaces**
| Origen | Interface  | Interface  | Destino |
| ------ | ------------- | ------------ | ------- |
| MSW3   | f0/3          | g1/0/4       | MSW Edificio 1 |
| MSW3   | g0/1          | g1/0/5       | MSW Edificio 1 |
| MSW3   | g0/2          | g1/0/6       | MSW Edificio 1 |
| MSW3   | f0/1          | f0/1         | MSW4 |
| MSW3   | f0/2          | f0/2         | MSW5 |

**Comandos**
```js
// VTP
vtp domain g18
vtp password admin
vtp mode client

// Configuramos VLANS
conf t
vlan 10
name naranja18
vlan 20
name verde18
exit

// Configuramos conexion hacia MSW4 y MSW5 (f0/1, f0/2) y hacia MSW Edificio 1 (g0/1, g0/2 y f0/3)
interface range fastEthernet0/1-3, gigabitEthernet0/1-2
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
no shutdown
exit

// Configuramos LACP hacia MSW Edificio 1
interface range gigabitEthernet0/1-2, fastEthernet0/3
channel-group 3 mode active
exit

interface port-channel 3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit

//configurar VRRP
interface vlan 10
standby 1 ip 192.168.10.1
standby 1 priority 110
standby 1 preempt
no shutdown
exit

// Configuramos VRRP para la VLAN 20
interface vlan 20
standby 2 ip 192.168.20.1
standby 2 priority 110
standby 2 preempt
no shutdown
exit

// Habilitamos el enrutamiento IP
ip routing

// Configuramos EIGRP (cambia "100" al número de proceso de EIGRP que desees)
router eigrp 100
network 192.168.10.0 0.0.0.255
network 192.168.20.0 0.0.0.255
no auto-summary
exit
end
```

#### **MSW 4, 5**
**Interfaces**
| Origen | Interface  | Interface  | Destino  |
| ------ | ------------- | ------------ | -------  |
| MSW4   | g0/1          | f0/1         | MSW3    |
| MSW4   | g0/2          | g0/2         | MSW6    |

**Comandos**
```js
conf t

// Configuramos VTP
vtp domain g18
vtp password admin
vtp mode client

// Configuramos VLANS
conf t
vlan 10
name naranja18
vlan 20
name verde18
exit

// Se configuran interfaces
interface range gigabitEthernet0/1-2
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan all
no shutdown
exit

// Configuración de HSRP para la VLAN 10 (naranja18)
interface vlan 10
// ip address 192.168.10.2 255.255.255.0 //Para MSW4 es .2, para MSW5 es .3
// ip address 192.168.10.3 255.255.255.0 
standby 1 ip 192.168.10.1
standby 1 priority 100 // 101 para el active
standby 1 preempt
exit

// Configuración de HSRP para la VLAN 20 (verde18)
interface vlan 20
// ip address 192.168.20.2 255.255.255.0 //Para MSW4 es .2, para MSW5 es .3
// ip address 192.168.20.3 255.255.255.0 
standby 2 ip 192.168.20.1
standby 2 priority 100 // 101 para el active
standby 2 preempt
exit

// Configuración de EIGRP
// ip routing
// router eigrp 100
// network 192.168.10.0
// network 192.168.20.0
// no auto-summary
// exit
```
### Configuracion Switch Core Edificio 1
#### **MSW6**
**Interfaces**
| Origen | Interface  | Interface  | Destino  |
| ------ | ------------- | ------------ | -------  |
| MSW6   | g0/1          | g0/1         | MSW5    |
| MSW6   | g0/2          | g0/2         | MSW4    |
| MSW6   | f0/1          | g0/1         | SW1     |
| MSW6   | f0/2          | g0/1         | SW2     |
**Comandos**
```js
// Configuramos VTP
vtp domain g18
vtp password admin
vtp mode client

// Configuramos VLANS
vlan 10
name naranja18
vlan 20
name verde18

// VLAN 10 - Naranja
interface vlan 10
ip address 192.168.10.4 255.255.255.0
no shutdown
exit

// VLAN 20 - Verde
interface vlan 20
ip address 192.168.20.4 255.255.255.0
no shutdown
exit

// Configuración de las conexiones a MSW4 y MSW5 (g0/1 y g0/2)
interface range gigabitEthernet0/1-2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit

// Configuración de las conexiones hacia Switch1 (SW1) y Switch2 (SW2)
interface range fastEthernet0/1-2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit

// Configuración de enrutamiento EIGRP
ip routing
router eigrp 100
network 192.168.10.0
network 192.168.20.0
no auto-summary
exit

end
```
### Switches Normales
#### **SW 1** 
**Interfaces**
| Origen | Interface  | Interface  | Destino  |
| ------ | ------------- | ------------ | -------  |
| SW1   | g0/1          | f0/1         | MSW6    |
| SW1   | f0/11         | f0         | PC1    |
| SW1   | f0/12         | f0         | PC0    |

**Comandos**
```js
//Configuramos VLAN 10 Naranja18
conft t
vlan 10
name naranja18
exit
// Configuramos conexion hacia MSW6
interface gigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan all
exit
// Configuramos usuarios
interface range fastEthernet0/11-12
switchport mode access
switchport access vlan 10
```

#### **SW 2** 
**Interfaces**
| Origen | Interface  | Interface  | Destino  |
| ------ | ------------- | ------------ | -------  |
| SW1   | g0/1          | f0/2         | MSW6    |
| SW1   | f0/11         | f0         | PC1    |
| SW1   | f0/12         | f0         | PC0    |

**Comandos**
```js
//Configuramos VLANS 20 Verde18
vlan 20
name verde18
exit
// Configuramos conexion hacia MSW6
interface gigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan all
exit
// Configuramos usuarios
interface range fastEthernet0/11-12
switchport mode access
switchport access vlan 20
```



## **HSRP** 
Para la configuracion de las switch paralelas se hace esta configuracion 

### MSW5 (pasivo) y MSW4 (activo)
Esta es para el switch activo
```
Router (config) interface GigabitEthernet0/2
Router (config-if) #ena
Router (config-if) #standby 1 ip 192.168.1.254
Router (config-if) #standby 1 priority 150
Router (config-if) #standby 1 preempt
```
Este es para el pasivo
```
Router (config) int g0/2
Router (config-if) #ena
Router (config-if) #standby 1 ip 192.168.1.254
```

### MSW 6
Esta es la configuracion del switch "destino" la que conecta con los dos anteriores
```
Switch>ena
Switch#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#int loopback 1
Switch(config-if)#ip add 209.165.100.10 255.255.255.240
Switch(config-if)#no shut
Switch(config-if)#exit
Switch(config)#ip route 0.0.0.0 0.0.0.0 loopback 1
Switch(config)#router rip
Switch(config-router)#default-information originate
```
