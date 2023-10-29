# ISP1
REDES
| Subred         | IPv4 Inicio | IPv4 Fin     | Netmask         | Host. Total | RED |
| -------------- | ----------- | ------------ | --------------- | ---------- | --- |
| 10.0.0.0/24  | 10.0.0.0  | 10.0.0.255 | 255.255.255.0 | 256        | Red Interna   |
| 192.168.59.0/24  | 192.168.59.0  | 192.168.59.255  | 255.255.255.0 | 256        | Yota   |
| 192.168.59.0/26  | 192.168.59.0  | 192.168.59.63  | 255.255.255.192 | 64         | 100 Adif   |
| 192.168.59.64/26 | 192.168.59.64 | 192.168.59.127 | 255.255.255.192 | 64         | 101 Rosneft   |

## Router
g0/0/0 -> MSW
```js
// No initial config
enable
conf t
// INRTERFACES
// encender hacia MSW
interface g0/0/0
no shutdown
exit
// encender hacia Internet
interface g0/0/1
ip address 11.0.0.1 255.255.255.252 // porque es /32
no shutdown
exit
// ENRUTAMIENTOS
// enrutamiento vlan 10 interna
interface g0/0/0.10
encapsulation dot1Q 10
ip address 10.0.0.1 255.255.255.0
exit
// enrutamiento vlan 100 
interface g0/0/0.100
encapsulation dot1Q 100
ip address 192.168.59.1 255.255.255.192
exit
// enrutamiento vlan 101
interface g0/0/0.101
encapsulation dot1Q 101
ip address 192.168.59.65 255.255.255.192
exit
// enrutamiento de ip por dhcp server
interface range g0/0/0.100, g0/0/0.101
ip helper-address 10.0.0.3
exit
// ENRUTAMIENTO OSPF
router ospf 1
network 11.0.0.0 0.0.0.3 area 0
network 10.0.0.0 0.0.0.3 area 0
network 192.168.59.0 0.0.0.63 area 0
network 192.168.59.64 0.0.0.63 area 0
exit
```
## MSW
```js
enable
conf t
// VLANS
vlan 10
name interna
exit
vlan 100
name Adif
exit
vlan 101
name Rosneft
exit
// INTERFACES
// hacia router
interface g0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
// hacia server del ISP
interface g0/2
switchport mode access
switchport access vlan 10
no shutdown
exit
// LACP Adif
interface range f0/1-3
channel-group 1 mode active
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
// LACP Rosneft
interface range f0/4-6
channel-group 2 mode active
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
interface port-channel 2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
// Configuración de EIGRP para las 2 redes
ip routing
router eigrp 100
network 192.168.59.0 0.0.0.63
network 192.168.59.64 0.0.0.63
no auto-summary
exit
```
## Server
Configuramos servidor
Ajustes -> desktop -> ip -> static
| IP | Configuration |
| -- | -- |
| **Ip Address:** | 10.0.0.3 | 
| **Subnet Mask:** | 255.255.255.0 |
| ** Default Gateway:** | 10.0.0.1 | 
Ajustes -> Servicios -> DHCP ON
Usuarios Adif
| DHCP |  Services |
| -- | -- |
| **Pool Name:** | vlan-adif |
| ** Default Gateway:** | 192.168.59.1 | 
| **Start Ip Address:** | 192.168.59.10 |
| **Subnet Mask:** | 255.255.255.192 |
| **Maximum Number of Users:** | 50 |
ADD POOL
Usuarios Rosneft
| DHCP |  Services |
| -- | -- |
| **Pool Name:** | vlan-rosneft |
| ** Default Gateway:** | 192.168.59.65 | 
| **Start Ip Address:** | 192.168.59.75 |
| **Subnet Mask:** | 255.255.255.192 |
| **Maximum Number of Users:** | 50 |
ADD POOL
RED INTERNA
| DHCP |  Services |
| -- | -- |
| **Pool Name:** | serverPool |
| ** Default Gateway:** | 10.0.0.1 | 
| **Start Ip Address:** | 10.0.0.10 |
| **Subnet Mask:** | 255.255.255.0 |
| **Maximum Number of Users:** | 200 |
ADD POOL
### Switch Adif
```js
enable
conf t
// VLAN
vlan 100
name Adif
exit
// LACP Adif
interface range f0/1-3
channel-group 1 mode active
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
interface port-channel 1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
// usuarios
interface range f0/11-12
switchport mode access
switchport access vlan 100
no shutdown
exit
```
### Switch Rosneft
```js
enable
conf t
// VLAN
vlan 101
name Rosneft
exit
// LACP Adif
interface range f0/1-3
channel-group 2 mode active
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
interface port-channel 2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
no shutdown
exit
// usuarios
interface range f0/11-12
switchport mode access
switchport access vlan 101
no shutdown
exit
```
# Internet
Direccionamiento ISP1 = 11.0.0.0/30
Direccionamiento ISP2 = 11.0.0.4/30
| # |  Interfaces |
| -- | -- |
| **Hacia ISP1** | 11.0.0.2 - 255.255.255.252 |
| **Hacia ISP2** | 11.0.0.5 - 255.255.255.252 |
## Router
```js
enable
conf t
// INTERFACES
// hacia ISP1
interface g0/0/1
ip address 11.0.0.2 255.255.255.252
no shutdown
exit
// hacia ISP2
interface g0/0/0
ip address 11.0.0.5 255.255.255.252
no shutdown
exit
// ENRUTAMIENTO OSPF
router ospf 1
network 11.0.0.0 0.0.0.3 area 0
network 11.0.0.4 0.0.0.3 area 0
exit
```
# ISP2
## Router
Interna = VLAN 20 - 172.16.1.0/24
Usuario = VLAN 200 - 200.20.167.0/24
g0/0/1 -> MSW
```js
// No initial config
enable
conf t
// INRTERFACES
// encender hacia Internet
interface g0/0/1
ip address 11.0.0.6 255.255.255.252 // porque es /32
no shutdown
exit
// encender hacia MSW
interface g0/0/0
no shutdown
exit
// ENRUTAMIENTOS
// enrutamiento vlan 10 
interface g0/0/0.20
encapsulation dot1Q 20
ip address 172.16.1.1 255.255.255.0
exit
// enrutamiento vlan 200 
interface g0/0/0.200
encapsulation dot1Q 200
ip address 200.20.167.1 255.255.255.0
exit
// enrutamiento de ip por dhcp server
interface g0/0/0.200
ip helper-address 172.16.1.2
exit
// ENRUTAMIENTO OSPF
router ospf 1
network 11.0.0.4 0.0.0.3 area 0
network 172.16.1.0 0.0.0.3 area 0
network 200.20.167.0 0.0.0.255 area 0
exit
```
## MSW
```js
enable
conf t
// VLANS
vlan 20
exit
vlan 200
exit
// INTERFACES
// hacia router
interface g0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan all
// hacia Cloud y MSW USUARIOS
interface range f0/1, f0/3
switchport mode access
switchport access vlan 200 //usuarios
no shutdown
exit
// hacia server del ISP
interface f0/2
switchport mode access
switchport access vlan 20
no shutdown
exit
```
## Server
Configuramos servidor
Ajustes -> desktop -> ip -> static
| IP | Configuration |
| -- | -- |
| **Ip Address:** | 172.16.1.2 | 
| **Subnet Mask:** | 255.255.255.0 |
| ** Default Gateway:** | 172.16.1.1 | 
Ajustes -> Servicios -> DHCP ON
Usuarios
| DHCP |  Services |
| -- | -- |
| **Pool Name:** | vlanisp2 |
| ** Default Gateway:** | 200.20.167.1 | 
| **Start Ip Address:** | 200.20.167.10 |
| **Subnet Mask:** | 255.255.255.0 |
| **Maximum Number of Users:** | 200 |
ADD POOL
RED INTERNA
| DHCP |  Services |
| -- | -- |
| **Pool Name:** | vlaninterna |
| ** Default Gateway:** | 172.16.1.1 | 
| **Start Ip Address:** | 172.16.1.10 |
| **Subnet Mask:** | 255.255.255.0 |
| **Maximum Number of Users:** | 200 |
ADD POOL
```js
```
## Cloud
Ajustes -> Ethernet6 -> DSL
Ajustes -> DSL -> Modem4 <-> Eth6
Agregar
```js
```