# Laboratorio CCNA
## Índice y Estructura Principal
- [Requerimientos](#requirimientos)
- [Topología en GNS3](#topología-en-gns3)
- [Configuración Etherchannel](#etherchannel)
    * [L3SW1](#l3sw1)
    * [L3SW2](#l3sw2)
    * [L3SW3](#l3sw3)
    * [L3SW4](#l3sw4)
- [Configuración Trunk, VTP, VLANS y buenas prácticas de seguridad](#trunking-vtp-vlans-y-buenas-prácticas)
    * [Configuración Trunk](#1-trunking)      
      * [L3SW1](#l3sw1)
      * [L3SW2](#l3sw2)
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)
    * [Configuración VTP](#2-vtp)      
      * [L3SW1](#l3sw1)
      * [L3SW2](#l3sw2)
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)
    * [Configuración VLANS](#3-vlans)      
      * [L3SW1](#l3sw1)
      * [L3SW2](#l3sw2)
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)
    * [Configuración buenas prácticas](#buenas-prácticas-de-seguridad-en-los-enlaces-troncales)      
      * [L3SW1](#l3sw1)
      * [L3SW2](#l3sw2)
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)
- [Configuración Access, Port-Security y buenas prácticas de seguridad](#access-y-port-security)
    * [Configuración Access](#1-asignación-de-vlan-a-los-puertos-de-acceso)      
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)
    * [Configuración Port-Security](#2-port-security)      
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)
    * [Buenas prácticas de seguridad](#3-buenas-prácticas)      
      * [L3SW1](#l3sw1)
      * [L3SW2](#l3sw2)
      * [L3SW3](#l3sw3)
      * [L3SW4](#l3sw4)
      * [L2SW1](#l2sw1)
      * [L2SW2](#l2sw2)   
## Requirimientos
### 1. IOSv Cisco
### Switch Multilayer
-   [vios_l2-adventerprisek9-m.03.2017.qcow2](https://drive.google.com/file/d/12G5QRHiOa4h6RMr0lDzEOPTdKcp4n3vC/view?usp=sharing)
### Router
-   [vios-adventerprisek9-m.vmdk.SPA.156-2.T.qcow2](https://drive.google.com/file/d/1YSzYeAK6YH81Hvu_aKTLOoVfArRWH5po/view?usp=sharing)

### 2. Appliance GNS3
### Servers
-   [AAA](https://www.gns3.com/marketplace/appliances/aaa-2)
-   [DNS](https://www.gns3.com/marketplace/appliances/dns)
-   [Networkers' Toolkit](https://www.gns3.com/marketplace/appliances/networkers-toolkit)
### PCs 
-   [webterm](https://www.gns3.com/marketplace/appliances/webterm)
-   [ipterm](https://www.gns3.com/marketplace/appliances/ipterm) 

## Topología en GNS3
![](/imagenes/topologia.png)
## Etherchannel
![](/imagenes/etherchannel.png)
### L3SW1
```go
L3SW1#show cdp neighbors | b Device
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
L3SW2            Gig 1/0           134             R S I            Gig 1/0
L3SW2            Gig 1/1           174             R S I            Gig 1/1
L3SW3            Gig 0/0           149             R S I            Gig 1/0
L3SW3            Gig 0/1           154             R S I            Gig 1/1
L3SW4            Gig 0/3           166             R S I            Gig 1/1
L3SW4            Gig 0/2           135             R S I            Gig 1/0
```
> Este comando es muy importante porque nos permitirá identificar que interfaces locales están conectados hacias los demás switches multicapa.
-   Gig1/0-1 hacia L3SW2 mode on (Po6)
-   Gig0/0-1 hacia L3SW3 mode desirable (Po1)
-   Gig0/2-3 hacia L3SW4 mode active (Po3)
```go
L3SW1(config)#int range g0/0-1
L3SW1(config-if-range)#channel-group 1 mode desirable
L3SW1(config-if-range)#int range g0/2-3
L3SW1(config-if-range)#channel-group 3 mode active 
L3SW1(config-if-range)#int range g1/0-1
L3SW1(config-if-range)#channel-group 6 mode on     
L3SW1(config-if-range)#end
```
```go
L3SW1#show etherchannel summary | b Group
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Gi0/0(P)    Gi0/1(P)    
3      Po3(SU)         LACP      Gi0/2(P)    Gi0/3(P)    
6      Po6(SU)          -        Gi1/0(P)    Gi1/1(P)
```
> La combinación de las banderas (SU) nos indica que nuestro etherchannel a levantado correctamente y que está funcionando.
```go
L3SW1#show interfaces port-channel 1 | i BW
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec,
```
> La salida de este comando nos indíca que nuestro ancho de bando ahora es de 2Gbps.
### L3SW2
```go
L3SW2#show cdp neighbors | b Device
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
L3SW1            Gig 1/1           164             R S I            Gig 1/1
L3SW1            Gig 1/0           170             R S I            Gig 1/0
L3SW3            Gig 0/1           161             R S I            Gig 1/3
L3SW3            Gig 0/0           170             R S I            Gig 1/2
L3SW4            Gig 0/3           150             R S I            Gig 1/3
L3SW4            Gig 0/2           140             R S I            Gig 1/2
```
> Este comando es muy importante porque nos permitirá identificar que interfaces locales están conectados hacias los demás switches multicapa.
-   Gig1/0-1 hacia L3SW1 mode on (Po6)
-   Gig0/0-1 hacia L3SW3 mode active (Po2)
-   Gig0/2-3 hacia L3SW4 mode desirable (Po4)
```go
L3SW2(config)#int range g0/0-1
L3SW2(config-if-range)#channel-group 2 mode active
L3SW2(config-if-range)#int range g0/2-3
L3SW2(config-if-range)#channel-group 4 mode desirable
L3SW2(config-if-range)#int range g1/0-1
L3SW2(config-if-range)#channel-group 6 mode on     
L3SW2(config-if-range)#end
```
```go
L3SW2#show etherchannel summary | b Group
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
2      Po2(SU)         LACP      Gi0/0(P)    Gi0/1(P)    
4      Po4(SU)         PAgP      Gi0/2(P)    Gi0/3(P)    
6      Po6(SU)          -        Gi1/0(P)    Gi1/1(P)
```
> La combinación de las banderas (SU) nos indica que nuestro etherchannel a levantado correctamente y que está funcionando.
```go
L3SW2#show interfaces port-channel 2 | i BW
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec,
```
> La salida de este comando nos indíca que nuestro ancho de bando ahora es de 2Gbps.
### L3SW3
```go
L3SW3#show cdp neighbors | b Device
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
L3SW2            Gig 1/3           160             R S I            Gig 0/1
L3SW2            Gig 1/2           159             R S I            Gig 0/0
L3SW1            Gig 1/0           178             R S I            Gig 0/0
L3SW1            Gig 1/1           128             R S I            Gig 0/1
L3SW4            Gig 0/3           150             R S I            Gig 0/3
L3SW4            Gig 0/2           147             R S I            Gig 0/2
```
> Este comando es muy importante porque nos permitirá identificar que interfaces locales están conectados hacias los demás switches multicapa.
-   Gig1/2-3 hacia L3SW2 mode active (Po2)
-   Gig1/0-1 hacia L3SW1 mode desirable (Po1)
-   Gig0/2-3 hacia L3SW4 mode on (Po5)
```go
L3SW3(config)#int range g1/0-1
L3SW3(config-if-range)#channel-group 1 mode desirable
L3SW3(config-if-range)#int range g1/2-3
L3SW3(config-if-range)#channel-group 2 mode active
L3SW3(config-if-range)#int range g0/2-3
L3SW3(config-if-range)#channel-group 5 mode on      
L3SW3(config-if-range)#end
```
```go
L3SW3#show etherchannel summary | b Group
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Gi1/0(P)    Gi1/1(P)    
2      Po2(SU)         LACP      Gi1/2(P)    Gi1/3(P)    
5      Po5(SU)          -        Gi0/2(P)    Gi0/3(P)
```
> La combinación de las banderas (SU) nos indica que nuestro etherchannel a levantado correctamente y que está funcionando.
```go
L3SW3#show interfaces port-channel 1 | i BW
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec,
```
> La salida de este comando nos indíca que nuestro ancho de bando ahora es de 2Gbps.
### L3SW3
```go
L3SW4#show cdp neighbors | b Device
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
L3SW3            Gig 0/2           162             R S I            Gig 0/2
L3SW3            Gig 0/3           174             R S I            Gig 0/3
L3SW2            Gig 1/3           132             R S I            Gig 0/3
L3SW2            Gig 1/2           166             R S I            Gig 0/2
L3SW1            Gig 1/1           179             R S I            Gig 0/3
L3SW1            Gig 1/0           141             R S I            Gig 0/2
```
> Este comando es muy importante porque nos permitirá identificar que interfaces locales están conectados hacias los demás switches multicapa.
-   Gig0/2-3 hacia L3SW3 mode on (Po5)
-   Gig1/2-3 hacia L3SW2 mode desirable (Po4)
-   Gig1/0-1 hacia L3SW1 mode active (Po3)
```go
L3SW4(config)#int range g1/0-1
L3SW4(config-if-range)#channel-group 3 mode active
L3SW4(config-if-range)#int range g1/2-3
L3SW4(config-if-range)#channel-group 4 mode desirable 
L3SW4(config-if-range)#int range g0/2-3
L3SW4(config-if-range)#channel-group 5 mode on      
L3SW4(config-if-range)#end
```
```go
L3SW4#show etherchannel summary | b Group
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
3      Po3(SU)         LACP      Gi1/0(P)    Gi1/1(P)    
4      Po4(SU)         PAgP      Gi1/2(P)    Gi1/3(P)    
5      Po5(SU)          -        Gi0/2(P)    Gi0/3(P)
```
> La combinación de las banderas (SU) nos indica que nuestro etherchannel a levantado correctamente y que está funcionando.
```go
L3SW4#show interfaces port-channel 3 | i BW
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec,
```
> La salida de este comando nos indíca que nuestro ancho de bando ahora es de 2Gbps.

## Trunking, VTP, VLANS y buenas prácticas
![](/imagenes/trunk.png)
### 1. Trunking
### L3SW1
```go
L3SW1(config)#int port-channel 1
L3SW1(config-if)#switchport trunk encapsulation dot1q 
L3SW1(config-if)#switchport mode trunk 
L3SW1(config-if)#switchport nonegotiate
L3SW1(config-if)#int port-channel 3
L3SW1(config-if)#switchport trunk encapsulation dot1q 
L3SW1(config-if)#switchport mode trunk 
L3SW1(config-if)#switchport nonegotiate
L3SW1(config-if)#end
```
```go
L3SW1#show interfaces trunk       

Port        Mode             Encapsulation  Status        Native vlan
Po1         on               802.1q         trunking      1
Po3         on               802.1q         trunking      1

Port        Vlans allowed on trunk
Po1         1-4094
Po3         1-4094
```
### L3SW2
```go
L3SW2(config)#int port-channel 2
L3SW2(config-if)#switchport trunk encapsulation dot1q 
L3SW2(config-if)#switchport mode trunk 
L3SW2(config-if)#switchport nonegotiate
L3SW2(config-if)#int port-channel 4
L3SW2(config-if)#switchport trunk encapsulation dot1q 
L3SW2(config-if)#switchport mode trunk 
L3SW2(config-if)#switchport nonegotiate
L3SW2(config-if)#end
```
```go
L3SW2#show interfaces trunk 

Port        Mode             Encapsulation  Status        Native vlan
Po2         on               802.1q         trunking      1
Po4         on               802.1q         trunking      1

Port        Vlans allowed on trunk
Po2         1-4094
Po4         1-4094
```
### L3SW3
```go
L3SW3(config)#int port-channel 1
L3SW3(config-if)#switchport trunk encapsulation dot1q 
L3SW3(config-if)#switchport mode trunk 
L3SW3(config-if)#switchport nonegotiate
L3SW3(config-if)#int port-channel 2
L3SW3(config-if)#switchport trunk encapsulation dot1q 
L3SW3(config-if)#switchport mode trunk 
L3SW3(config-if)#switchport nonegotiate
L3SW3(config-if)#int port-channel 5
L3SW3(config-if)#switchport trunk encapsulation dot1q 
L3SW3(config-if)#switchport mode trunk 
L3SW3(config-if)#switchport nonegotiate
L3SW3(config-if)#int range g0/0-1
L3SW3(config-if-range)#switchport mode trunk 
L3SW3(config-if-range)#switchport nonegotiate
L3SW3(config-if-range)#end
```
```go
L3SW3#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
Gi0/1       on               802.1q         trunking      1
Po1         on               802.1q         trunking      1
Po2         on               802.1q         trunking      1
Po5         on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       1-4094
Gi0/1       1-4094
Po1         1-4094
Po2         1-4094
Po5         1-4094
```
### L3SW4
```go
L3SW4(config)#int port-channel 3
L3SW4(config-if)#switchport trunk encapsulation dot1q 
L3SW4(config-if)#switchport mode trunk 
L3SW4(config-if)#switchport nonegotiate
L3SW4(config-if)#int port-channel 4
L3SW4(config-if)#switchport trunk encapsulation dot1q 
L3SW4(config-if)#switchport mode trunk 
L3SW4(config-if)#switchport nonegotiate
L3SW4(config-if)#int port-channel 5
L3SW4(config-if)#switchport trunk encapsulation dot1q 
L3SW4(config-if)#switchport mode trunk 
L3SW4(config-if)#switchport nonegotiate
L3SW4(config-if)#int range g0/0-1
L3SW4(config-if-range)#switchport trunk encapsulation dot1q 
L3SW4(config-if-range)#switchport mode trunk 
L3SW4(config-if-range)#switchport nonegotiate
L3SW4(config-if-range)#end
```
```go
L3SW4#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
Gi0/1       on               802.1q         trunking      1
Po3         on               802.1q         trunking      1
Po4         on               802.1q         trunking      1
Po5         on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       1-4094
Gi0/1       1-4094
Po3         1-4094
Po4         1-4094
Po5         1-4094
```
### L2SW1
```go
L2SW1(config)#int range g0/0-1
L2SW1(config-if-range)#switchport trunk encapsulation dot1q 
L2SW1(config-if-range)#switchport mode trunk 
L2SW1(config-if-range)#switchport nonegotiate
L2SW1(config-if-range)#end
```
```go
L2SW1#show interfaces trunk 

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
Gi0/1       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       1-4094
Gi0/1       1-4094
```
### L2SW2
```go
L2SW2(config)#int range g0/0-1
L2SW2(config-if-range)#switchport trunk encapsulation dot1q 
L2SW2(config-if-range)#switchport mode trunk 
L2SW2(config-if-range)#switchport nonegotiate
L2SW2(config-if-range)#end
```
```go
L2SW2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
Gi0/1       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       1-4094
Gi0/1       1-4094
```
### 2. VTP
- VTP Server: L3SW3
- VTP Client: L3SW1, L3SW2, L3SW4, L2SW1 y L2SW2
- VTP domain: cisco.com
- VTP password: vtppass
- Habilitamos VTP Prunning
### L3SW3
```go
L3SW3(config)#vtp mode server
L3SW3(config)#vtp domain cisco.com
L3SW3(config)#vtp password vtppass
L3SW3(config)#vtp version 2
L3SW3(config)#end
```
```go
L3SW3#show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : cisco.com
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0c65.cddb.8000
Configuration last modified by 0.0.0.0 at 3-15-22 20:36:12
Local updater ID is 0.0.0.0 (no valid interface found)

Feature VLAN:
--------------
VTP Operating Mode                : Server
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 13
MD5 digest                        : 0x7D 0x43 0xD0 0xF6 0xEA 0x8C 0xD3 0x22 
                                    0x7D 0x76 0x7F 0x02 0xAD 0x5A 0x6B 0x8F
```
### L3SW1
```go
L3SW1(config)#vtp mode client
L3SW1(config)#vtp domain cisco.com
L3SW1(config)#vtp password vtppass
L3SW1(config)#end
```
```go
L3SW1#show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : cisco.com
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0c08.9518.8000
Configuration last modified by 0.0.0.0 at 3-15-22 20:36:12

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 13
MD5 digest                        : 0x7D 0x43 0xD0 0xF6 0xEA 0x8C 0xD3 0x22 
                                    0x7D 0x76 0x7F 0x02 0xAD 0x5A 0x6B 0x8F
```
### L3SW2
```go
L3SW2(config)#vtp mode client
L3SW2(config)#vtp domain cisco.com
L3SW2(config)#vtp password vtppass
L3SW2(config)#end
```
```go
L3SW2#show vtp status 
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : cisco.com
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0c04.9562.8000
Configuration last modified by 0.0.0.0 at 3-15-22 20:36:12

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 13
MD5 digest                        : 0x7D 0x43 0xD0 0xF6 0xEA 0x8C 0xD3 0x22 
                                    0x7D 0x76 0x7F 0x02 0xAD 0x5A 0x6B 0x8F
```
### L3SW4
```go
L3SW4(config)#vtp mode client
L3SW4(config)#vtp domain cisco.com
L3SW4(config)#vtp password vtppass
L3SW4(config)#end
```
```go
L3SW4#show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : cisco.com
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0cc4.6c44.8000
Configuration last modified by 0.0.0.0 at 3-15-22 20:36:12

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 13
MD5 digest                        : 0x7D 0x43 0xD0 0xF6 0xEA 0x8C 0xD3 0x22 
                                    0x7D 0x76 0x7F 0x02 0xAD 0x5A 0x6B 0x8F
```
### L2SW1
```go
L2SW1(config)#vtp mode client
L2SW1(config)#vtp domain cisco.com
L2SW1(config)#vtp password vtppass
L2SW1(config)#end
```
```go
L2SW1#show vtp status 
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : cisco.com
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0ce4.8b9c.8000
Configuration last modified by 0.0.0.0 at 3-15-22 20:36:12

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 13
MD5 digest                        : 0x7D 0x43 0xD0 0xF6 0xEA 0x8C 0xD3 0x22 
                                    0x7D 0x76 0x7F 0x02 0xAD 0x5A 0x6B 0x8F
```
### L2SW2
```go
L2SW2(config)#vtp mode client
L2SW2(config)#vtp domain cisco.com
L2SW2(config)#vtp password vtppass
L2SW2(config)#end
```
```go
L2SW2#show vtp status
VTP Version capable             : 1 to 3
VTP version running             : 2
VTP Domain Name                 : cisco.com
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0c48.5912.8000
Configuration last modified by 0.0.0.0 at 3-15-22 20:36:12

Feature VLAN:
--------------
VTP Operating Mode                : Client
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 9
Configuration Revision            : 13
MD5 digest                        : 0x7D 0x43 0xD0 0xF6 0xEA 0x8C 0xD3 0x22 
                                    0x7D 0x76 0x7F 0x02 0xAD 0x5A 0x6B 0x8F
```
### 3. VLANS
- VLAN 10: SALES
- VLAN 20: TAC
- VLAN 30: SERVERS
- VLAN 99: MGMNT
> Creamos las VLANS en el servidor VTP y por medio de este protocolo se propagarán las Vlans hacia los clientes VTP.
### L3SW3
```go
L3SW3(config)#vlan 10
L3SW3(config-vlan)#name SALES
L3SW3(config-vlan)#vlan 20
L3SW3(config-vlan)#name TAC
L3SW3(config-vlan)#vlan 30
L3SW3(config-vlan)#name SERVERS
L3SW3(config-vlan)#vlan 99
L3SW3(config-vlan)#name MGMNT
L3SW3(config-vlan)#end
```
```go
L3SW3#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi2/0, Gi2/1, Gi2/2, Gi2/3
                                                Gi3/0, Gi3/1, Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active    
```
### L3SW1
```go
L3SW1#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/2, Gi1/3, Gi2/0, Gi2/1
                                                Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3, Po6
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active    
```
### L3SW2
```go
L3SW2#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/2, Gi1/3, Gi2/0, Gi2/1
                                                Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3, Po6
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active     
```
### L3SW4
```go
L3SW4#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi2/0, Gi2/1, Gi2/2, Gi2/3
                                                Gi3/0, Gi3/1, Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active    
```
### L2SW1
```go
L2SW1#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3, Gi1/0, Gi1/1
                                                Gi1/2, Gi1/3, Gi2/0, Gi2/1
                                                Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active    
```
### L2SW2
```go
L2SW2#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3, Gi1/0, Gi1/1
                                                Gi1/2, Gi1/3, Gi2/0, Gi2/1
                                                Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active    
```
> No es válido crear VLANS en los clientes VTP y si lo intentamos se genera un error.
```go
L2SW1(config)#vlan 50
VTP VLAN configuration not allowed when device is in CLIENT mode.
```
### Buenas prácticas de seguridad en los enlaces troncales
### L3SW1
```go
L3SW1(config)#int port-channel 1
L3SW1(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW1(config-if)#switchport trunk native vlan 99
L3SW1(config-if)#int port-channel 3
L3SW1(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW1(config-if)#switchport trunk native vlan 99
L3SW1(config-if)#end
```
```go
L3SW1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Po1         on               802.1q         trunking      99
Po3         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         10,20,30,99
Po3         10,20,30,99
```
### L3SW2
```go
L3SW2(config)#int port-channel 2
L3SW2(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW2(config-if)#switchport trunk native vlan 99
L3SW2(config-if)#int port-channel 4
L3SW2(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW2(config-if)#switchport trunk native vlan 99
L3SW2(config-if)#end
```
```go
L3SW2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Po2         on               802.1q         trunking      99
Po4         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Po2         10,20,30,99
Po4         10,20,30,99
```
### L3SW3
```go
L3SW3(config)#int port-channel 1
L3SW3(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW3(config-if)#switchport trunk native vlan 99
L3SW3(config-if)#int port-channel 2
L3SW3(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW3(config-if)#switchport trunk native vlan 99
L3SW3(config-if)#int port-channel 5
L3SW3(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW3(config-if)#switchport trunk native vlan 99
L3SW3(config-if)#int range g0/0-1
L3SW3(config-if-range)#switchport trunk allowed vlan 10,20,30,99
L3SW3(config-if-range)#switchport trunk native vlan 99
L3SW3(config-if-range)#end
```
```go
L3SW3#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99
Gi0/1       on               802.1q         trunking      99
Po1         on               802.1q         trunking      99
Po2         on               802.1q         trunking      99
Po5         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,99
Gi0/1       10,20,30,99
Po1         10,20,30,99
Po2         10,20,30,99
Po5         10,20,30,99
```
### L3SW4
```go
L3SW4(config)#int port-channel 3
L3SW4(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW4(config-if)#switchport trunk native vlan 99
L3SW4(config-if)#int port-channel 4
L3SW4(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW4(config-if)#switchport trunk native vlan 99
L3SW4(config-if)#int port-channel 5
L3SW4(config-if)#switchport trunk allowed vlan 10,20,30,99
L3SW4(config-if)#switchport trunk native vlan 99
L3SW4(config-if)#int range g0/0-1
L3SW4(config-if-range)#switchport trunk allowed vlan 10,20,30,99
L3SW4(config-if-range)#switchport trunk native vlan 99
L3SW4(config-if-range)#end
```
```go
L3SW4#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99
Gi0/1       on               802.1q         trunking      99
Po3         on               802.1q         trunking      99
Po4         on               802.1q         trunking      99
Po5         on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,99
Gi0/1       10,20,30,99
Po3         10,20,30,99
Po4         10,20,30,99
Po5         10,20,30,99
```
### L2SW1
```go
L2SW1(config)#int range g0/0-1
L2SW1(config-if-range)#switchport trunk allowed vlan 10,20,30,99
L2SW1(config-if-range)#switchport trunk native vlan 99
L2SW1(config-if-range)#end
```
```go
L2SW1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99
Gi0/1       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,99
Gi0/1       10,20,30,99
```
### L2SW2
```go
L2SW2(config)#int range g0/0-1
L2SW2(config-if-range)#switchport trunk allowed vlan 10,20,30,99
L2SW2(config-if-range)#switchport trunk native vlan 99
L2SW2(config-if-range)#end
```
```go
L2SW2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99
Gi0/1       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,99
Gi0/1       10,20,30,99
```
> Asignamos como vlan nativa la vlan 99 y permitimos que solo pasen por los enlaces troncales las vlans que creamos (10, 20, 30 y 99)
### Access, Port-security y buenas prácticas
![](/imagenes/access.png)
### 1. Asignación de vlan a los puertos de acceso
### L3SW3
```go
L3SW3(config)#int range g2/0-1
L3SW3(config-if-range)#switchport mode access
L3SW3(config-if-range)#switchport access vlan 30
L3SW3(config-if-range)#end
```
```go
L3SW3#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    Gi2/0, Gi2/1
99   MGMNT                            active    
```
### L3SW4
```go
L3SW4(config)#int range g2/0-1
L3SW4(config-if-range)#switchport mode access
L3SW4(config-if-range)#switchport access vlan 30
L3SW4(config-if-range)#end
```
```go
L3SW4#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    
30   SERVERS                          active    Gi2/0, Gi2/1
99   MGMNT                            active  
```
### L2SW1
```go
L2SW1(config)#int g0/2
L2SW1(config-if)#switchport mode access
L2SW1(config-if)#switchport access vlan 99
L2SW1(config-if)#int g0/3
L2SW1(config-if)#switchport mode access
L2SW1(config-if)#switchport access vlan 10
L2SW1(config-if)#int g1/0
L2SW1(config-if)#switchport mode access
L2SW1(config-if)#switchport access vlan 10
L2SW1(config-if)#end
```
```go
L2SW1#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/1, Gi1/2, Gi1/3, Gi2/0
                                                Gi2/1, Gi2/2, Gi2/3, Gi3/0
                                                Gi3/1, Gi3/2, Gi3/3
10   SALES                            active    Gi0/3, Gi1/0
20   TAC                              active    
30   SERVERS                          active    
99   MGMNT                            active    Gi0/2
```
### L2SW2
```go
L2SW2(config)#int g0/2
L2SW2(config-if)#switchport mode access
L2SW2(config-if)#switchport access vlan 99
L2SW2(config-if)#int g0/3
L2SW2(config-if)#switchport mode access
L2SW2(config-if)#switchport access vlan 20
L2SW2(config-if)#int g1/0
L2SW2(config-if)#switchport mode access
L2SW2(config-if)#switchport access vlan 20
L2SW2(config-if)#end
```
```go
L2SW2#show vlan brief | e unsup

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/1, Gi1/2, Gi1/3, Gi2/0
                                                Gi2/1, Gi2/2, Gi2/3, Gi3/0
                                                Gi3/1, Gi3/2, Gi3/3
10   SALES                            active    
20   TAC                              active    Gi0/3, Gi1/0
30   SERVERS                          active    
99   MGMNT                            active    Gi0/2
```
### 2. Port-Security
### L3SW3
```go
L3SW3(config)#int range g2/0-1
L3SW3(config-if-range)#switchport port-security 
L3SW3(config-if-range)#switchport port-security maximum 1
L3SW3(config-if-range)#switchport port-security mac-address sticky 
L3SW3(config-if-range)#switchport port-security violation restrict 
L3SW3(config-if-range)#end 
```
```go
L3SW3#show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
      Gi2/0              1            0                  0         Restrict
      Gi2/1              1            0                  0         Restrict
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 4096
```
### L3SW4
```go
L3SW4(config)#int range g2/0-1
L3SW4(config-if-range)#switchport port-security 
L3SW4(config-if-range)#switchport port-security maximum 1
L3SW4(config-if-range)#switchport port-security mac-address sticky 
L3SW4(config-if-range)#switchport port-security violation restrict 
L3SW4(config-if-range)#end
```
```go
L3SW4#show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
      Gi2/0              1            0                  0         Restrict
      Gi2/1              1            0                  0         Restrict
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 4096
```
### L2SW1
```go
L2SW1(config)#int range g0/2
L2SW1(config-if-range)#switchport port-security 
L2SW1(config-if-range)#switchport port-security maximum 2
L2SW1(config-if-range)#switchport port-security mac-address sticky 
L2SW1(config-if-range)#switchport port-security violation restrict 
L2SW1(config-if-range)#int range g0/3
L2SW1(config-if-range)#switchport port-security 
L2SW1(config-if-range)#switchport port-security maximum 2
L2SW1(config-if-range)#switchport port-security mac-address sticky 
L2SW1(config-if-range)#switchport port-security violation restrict
L2SW1(config-if-range)#int range g1/0
L2SW1(config-if-range)#switchport port-security 
L2SW1(config-if-range)#switchport port-security maximum 2
L2SW1(config-if-range)#switchport port-security mac-address sticky 
L2SW1(config-if-range)#switchport port-security violation restrict
L2SW1(config-if-range)#end
```
```go
L2SW1#show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
      Gi0/2              2            0                  0         Restrict
      Gi0/3              2            0                  0         Restrict
      Gi1/0              2            0                  0         Restrict
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 4096
```
### L2SW2
```go
L2SW2(config)#int range g0/2
L2SW2(config-if-range)#switchport port-security 
L2SW2(config-if-range)#switchport port-security maximum 2
L2SW2(config-if-range)#switchport port-security mac-address sticky 
L2SW2(config-if-range)#switchport port-security violation restrict 
L2SW2(config-if-range)#int range g0/3
L2SW2(config-if-range)#switchport port-security 
L2SW2(config-if-range)#switchport port-security maximum 2
L2SW2(config-if-range)#switchport port-security mac-address sticky 
L2SW2(config-if-range)#switchport port-security violation restrict
L2SW2(config-if-range)#int range g1/0
L2SW2(config-if-range)#switchport port-security 
L2SW2(config-if-range)#switchport port-security maximum 2
L2SW2(config-if-range)#switchport port-security mac-address sticky 
L2SW2(config-if-range)#switchport port-security violation restrict
L2SW2(config-if-range)#end
```
```go
L2SW2#show port-security
Secure Port  MaxSecureAddr  CurrentAddr  SecurityViolation  Security Action
                (Count)       (Count)          (Count)
---------------------------------------------------------------------------
      Gi0/2              2            0                  0         Restrict
      Gi0/3              2            0                  0         Restrict
      Gi1/0              2            0                  0         Restrict
---------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 4096
```
### 3. Buenas prácticas
> Apagamos todas las interfaces que están sin uso manualmente con el comando shutdown.
### L3SW1
```go
L3SW1(config)#int g1/3
L3SW1(config-if)#shutdown
L3SW1(config-if)#int range g2/0-3
L3SW1(config-if-range)#shutdown
L3SW1(config-if-range)#int range g3/0-3
L3SW1(config-if-range)#shutdown
L3SW1(config-if-range)#end
```
```go
L3SW1#show ip int br | i down
GigabitEthernet1/3     unassigned      YES unset  administratively down down    
GigabitEthernet2/0     unassigned      YES unset  administratively down down    
GigabitEthernet2/1     unassigned      YES unset  administratively down down    
GigabitEthernet2/2     unassigned      YES unset  administratively down down    
GigabitEthernet2/3     unassigned      YES unset  administratively down down    
GigabitEthernet3/0     unassigned      YES unset  administratively down down    
GigabitEthernet3/1     unassigned      YES unset  administratively down down    
GigabitEthernet3/2     unassigned      YES unset  administratively down down    
GigabitEthernet3/3     unassigned      YES unset  administratively down down 
```
### L3SW2
```go
L3SW2(config)#int g1/3
L3SW2(config-if)#shutdown
L3SW2(config-if)#int range g2/0-3
L3SW2(config-if-range)#shutdown
L3SW2(config-if-range)#int range g3/0-3
L3SW2(config-if-range)#shutdown
L3SW2(config-if-range)#end
```
```go
L3SW2#show ip int br | i down
GigabitEthernet1/3     unassigned      YES unset  administratively down down    
GigabitEthernet2/0     unassigned      YES unset  administratively down down    
GigabitEthernet2/1     unassigned      YES unset  administratively down down    
GigabitEthernet2/2     unassigned      YES unset  administratively down down    
GigabitEthernet2/3     unassigned      YES unset  administratively down down    
GigabitEthernet3/0     unassigned      YES unset  administratively down down    
GigabitEthernet3/1     unassigned      YES unset  administratively down down    
GigabitEthernet3/2     unassigned      YES unset  administratively down down    
GigabitEthernet3/3     unassigned      YES unset  administratively down down 
```
### L3SW3
```go
L3SW3(config)#int range g2/2-3
L3SW3(config-if-range)#shutdown
L3SW3(config-if-range)#int range g3/0-3
L3SW3(config-if-range)#shutdown
L3SW3(config-if-range)#end
```
```go
L3SW3#show ip int br | i down
GigabitEthernet2/2     unassigned      YES unset  administratively down down    
GigabitEthernet2/3     unassigned      YES unset  administratively down down    
GigabitEthernet3/0     unassigned      YES unset  administratively down down    
GigabitEthernet3/1     unassigned      YES unset  administratively down down    
GigabitEthernet3/2     unassigned      YES unset  administratively down down    
GigabitEthernet3/3     unassigned      YES unset  administratively down down
```
### L3SW4
```go
L3SW4(config)#int range g2/2-3
L3SW4(config-if-range)#shutdown
L3SW4(config-if-range)#int range g3/0-3
L3SW4(config-if-range)#shutdown
L3SW4(config-if-range)#end
```
```go
L3SW4#show ip int br | i down
GigabitEthernet2/2     unassigned      YES unset  administratively down down    
GigabitEthernet2/3     unassigned      YES unset  administratively down down    
GigabitEthernet3/0     unassigned      YES unset  administratively down down    
GigabitEthernet3/1     unassigned      YES unset  administratively down down    
GigabitEthernet3/2     unassigned      YES unset  administratively down down    
GigabitEthernet3/3     unassigned      YES unset  administratively down down
```
### L2SW1
```go
L2SW1(config)#int range g1/1-3
L2SW1(config-if-range)#shutdown
L2SW1(config-if-range)#int range g2/0-3
L2SW1(config-if-range)#shutdown
L2SW1(config-if-range)#int range g3/0-3
L2SW1(config-if-range)#shutdown
L2SW1(config-if-range)#end
```
```go
L2SW1#show ip int br | i down
GigabitEthernet1/1     unassigned      YES unset  administratively down down    
GigabitEthernet1/2     unassigned      YES unset  administratively down down    
GigabitEthernet1/3     unassigned      YES unset  administratively down down    
GigabitEthernet2/0     unassigned      YES unset  administratively down down    
GigabitEthernet2/1     unassigned      YES unset  administratively down down    
GigabitEthernet2/2     unassigned      YES unset  administratively down down    
GigabitEthernet2/3     unassigned      YES unset  administratively down down    
GigabitEthernet3/0     unassigned      YES unset  administratively down down    
GigabitEthernet3/1     unassigned      YES unset  administratively down down    
GigabitEthernet3/2     unassigned      YES unset  administratively down down    
GigabitEthernet3/3     unassigned      YES unset  administratively down down
```
### L2SW2
```go
L2SW2(config)#int range g1/1-3
L2SW2(config-if-range)#shutdown
L2SW2(config-if-range)#int range g2/0-3
L2SW2(config-if-range)#shutdown
L2SW2(config-if-range)#int range g3/0-3
L2SW2(config-if-range)#shutdown
L2SW2(config-if-range)#end
```
```go
L2SW2#show ip int br | i down
GigabitEthernet1/1     unassigned      YES unset  administratively down down    
GigabitEthernet1/2     unassigned      YES unset  administratively down down    
GigabitEthernet1/3     unassigned      YES unset  administratively down down    
GigabitEthernet2/0     unassigned      YES unset  administratively down down    
GigabitEthernet2/1     unassigned      YES unset  administratively down down    
GigabitEthernet2/2     unassigned      YES unset  administratively down down    
GigabitEthernet2/3     unassigned      YES unset  administratively down down    
GigabitEthernet3/0     unassigned      YES unset  administratively down down    
GigabitEthernet3/1     unassigned      YES unset  administratively down down    
GigabitEthernet3/2     unassigned      YES unset  administratively down down    
GigabitEthernet3/3     unassigned      YES unset  administratively down down 
```