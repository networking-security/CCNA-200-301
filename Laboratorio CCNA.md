# Laboratorio CCNA
## Índice y Estructura Principal
- [Requerimientos](#requirimientos)
- [Topología en GNS3](#topología-en-gns3)
- [Configuración Etherchannel](#etherchannel)
    * [L3SW1](#l3sw1)
    * [L3SW2](#l3sw2)
    * [L3SW3](#l3sw3)
    * [L3SW4](#l3sw4)
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