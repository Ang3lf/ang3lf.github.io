---
title: GRE
layout: default
---
*Generic Routing Encapsulation* es un protocolo que encapsula paquetes ip y los transporta a tráves de varias redes de forma encriptada, esto es posible debido a que encapsula paquetes ip dentro de otro paquete para mantener la integridad de los datos que viajan, básicamente es una forma de hacer una conexión directa punto a punto en la red, este protocolo puede funcionar con varios protocolos de red.

GRE trabaja en conjunto con túneles, un túnel es una interfaz interna que se utilizan en las VPN basadas en rutas para enrutar el tráfico de texto sin formato a un túnel VPN con la finalidad de mantener la integridad de los datos y así como se puede usar para fines productivos también este protocolo puede ser usado por personas que buscan ser anónimos para llevar a cabo ataques de red debido a que no se sabe con exactitud la fuente de este.

### Configuración

La configuración de este protocolo será sobre equipos de la marca Huawei.

![w1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEikttSbZr7RLjAirXNKIaJ8AcFxlnNKnHOMvPQm8YgAqdFsTu1MPRCzYr6PJfZfwA7QnFKvjPmo_0uwU15U02hjF_P8U8Pq22JKc1lKQVrJnd7d9F-BefcvwAFN8wqe8YCEaXdWTjJwIXHBEGkBeWrXJ0qsg3nZ9WPHw7NAlLpF771l2Ltkz-pYQMPDbA/s1520/Diagrama%20en%20blanco%20(1).jpeg)

La configuración del túnel se hará entre R1 y R2, primero empezemos haciendo configuraciones de routing en los equipos para que haya comunicación entre PC1 y PC2

```html
<Core-1>undo terminal monitor 
Info: Current terminal monitor is off.
<Core-1>system-view 
Enter system view, return user view with return command.
[~Core-1]ip vpn-instance Test
[*Core-1-vpn-instance-Test]ipv4-family 	
[*Core-1-vpn-instance-Test-af-ipv4]route-distinguisher 1:1	
[*Core-1-vpn-instance-Test-af-ipv4]vpn-target 1:1 both 
[*Core-1-vpn-instance-Test-af-ipv4]commit
[~Core-1-vpn-instance-Test-af-ipv4]disp this
 ipv4-family
  route-distinguisher 1:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
return
```
Creamos una instancia de VPN en el equipo llamada test, le asignamos un router distinguisher para que otros equipos puedan unirse a esta misma instancia y se agrega las comunidades de la vpn

> También se puede agregar las comunidades así: *'vpn-target 1:1 export-extcommunity'* / *'vpn-target 1:1 import-extcommunity'*. Para ver nuestras configuraciones hechas podemos poner '*display current-configuration configuration vpn-instance Test'*

Estas mismas configuraciones serían para los otros equipos
```html
<Core-3>undo terminal monitor 
Info: Current terminal monitor is off.
<Core-3>disp current-configuration configuration vpn-instance Test
#
ip vpn-instance Test
 ipv4-family
  route-distinguisher 1:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
return

<Core-2>undo terminal monitor 
Info: Current terminal monitor is off.
<Core-2>system-view 
Enter system view, return user view with return command.
[~Core-2]disp curr conf vpn Test
#
ip vpn-instance Test
 ipv4-family
  route-distinguisher 1:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
return

<R1>disp curr conf vpn Test
#
ip vpn-instance Test
 ipv4-family
  route-distinguisher 1:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
return

<R2>disp curr conf vpn Test
#
ip vpn-instance Test
 ipv4-family
  route-distinguisher 1:1
  vpn-target 1:1 export-extcommunity
  vpn-target 1:1 import-extcommunity
#
return
```

#### Direccionamiento
```html
[~Core-1-Ethernet1/0/0]ip binding vpn-instance Test
[~Core-1-Ethernet1/0/0]ip address 10.180.17.2 255.255.255.248
[~Core-1-Ethernet1/0/0]description to-Core-3.Ethernet1/0/1
[~Core-1-Ethernet1/0/0]undo shutdown
[~Core-1-Ethernet1/0/0]commit
[~Core-1-Ethernet1/0/0]disp this
#
interface Ethernet1/0/0
 description to-Core-3.Ethernet1/0/1
 undo shutdown
 ip binding vpn-instance Test
 ip address 10.180.17.2 255.255.255.248
 dcn
 dcn mode vlan
#
return
[~Core-1]display current-configuration interface Ethernet 1/0/1
#
interface Ethernet1/0/1
 description to-R1.Ethernet0/0/0
 undo shutdown
 ip binding vpn-instance Test
 ip address 10.80.115.1 255.255.255.224
```
Se asocia la instancia vpn 'Test' a esta interfaz física, y estas configuraciones se hacen en los otros equipos con el direccionamiento que corresponda

```html
[~Core-3-Ethernet1/0/1]disp this
#
interface Ethernet1/0/1
 description to-Core-1.Ethernet1/0/0
 undo shutdown
 ip binding vpn-instance Test
 ip address 10.180.17.1 255.255.255.248
 dcn
 dcn mode vlan
#
return
<Core-3>disp curr int ethe 1/0/0
#
interface Ethernet1/0/0
 description to-Core-2.Ethernet1/0/0
 undo shutdown
 ip binding vpn-instance Test
 ip address 10.180.25.1 255.255.255.240

 <Core-2>disp curr int ethe1/0/0
#
interface Ethernet1/0/0
 description to-Core-3.Ethernet1/0/0
 undo shutdown
 ip binding vpn-instance Test
 ip address 10.180.25.2 255.255.255.240
<Core-2>disp curr int ethe1/0/1
#
interface Ethernet1/0/1
 description to-R2.Ethernet0/0/0
 undo shutdown
 ip binding vpn-instance Test
 ip address 10.80.224.33 255.255.255.224

 <R1>disp curr int ether 0/0/0
#
interface Ethernet0/0/0
 description to-Core-1.Ethernet1/0/1
 ip binding vpn-instance Test
 ip address 10.80.115.2 255.255.255.224
#
return
<R1>disp curr int ether 0/0/1
#
interface Ethernet0/0/1
 description to-client-PC1
 ip binding vpn-instance Test
 ip address 10.80.213.97 255.255.255.240
#
return

<R2>disp curr int ether0/0/0
interface Ethernet0/0/0
 description to-Core-2.Ethernet1/0/1
 ip binding vpn-instance Test
 ip address 10.80.224.34 255.255.255.224
#
return
<R2>disp curr int ether0/0/1
#
interface Ethernet0/0/1
 description to-client.PC2
 ip binding vpn-instance Test
 ip address 169.254.161.1 255.255.255.252
#
return
```

#### Enrutamiento
```html
[~Core-1]bgp 5
[*Core-1-bgp]ipv4-family vpn-instance Test 
[~Core-1-bgp-Test]router-id 2.2.2.2
[*Core-1-bgp-Test]peer 10.80.115.2 as-number 5
[*Core-1-bgp-Test]peer 10.180.17.1 as-number 10
[*Core-1-bgp-Test]import-route direct
[*Core-1-bgp-Test]commit
[~Core-1-bgp-Test]disp this
 #
 ipv4-family vpn-instance Test
  router-id 2.2.2.2
  import-route direct
  peer 10.80.115.2 as-number 5
  peer 10.180.17.1 as-number 10
#
return
```
Se configura BGP en los routers, en IBGP los routers Core-1, Core-2, R1, R2 y EBGP Core-1 hacia Core-2 y Core-3, se pone los peers con los *AS* (autonomous system) que correspondan
```html
<R1>disp curr | begin bgp
bgp 5
 #
 ipv4-family unicast
  undo synchronization
 #
 ipv4-family vpn-instance Test
  router-id 3.3.3.3
  network 10.80.213.96 255.255.255.240
  peer 10.80.115.1 as-number 5
#
```
Se forma la primera adyacencia entre Core-1 y R1
```html
[~Core-1-bgp-Test]disp bgp vpnv4 vpn Test peer
 
 BGP local router ID : 0.0.0.0
 Local AS number : 5

 VPN-Instance Test, Router ID 2.2.2.2:
 Total number of peers : 2                 Peers in established state : 1

  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State  Pr
efRcv
  10.80.115.2     4           5        3        6     0 00:01:38 Established    
    0
  10.180.17.1     4          10        0        0     0 00:02:10      Active    
    0
```
```html
<R2>disp curr | begin bgp
#
 #
 ipv4-family vpn-instance Test
  router-id 5.5.5.5
  network 169.254.161.0 255.255.255.252
  peer 10.80.224.33 as-number 5
#
return
```
```html
<Core-2>disp bgp vpnv4 vpn Test peer
 
 BGP local router ID : 0.0.0.0
 Local AS number : 5

 VPN-Instance Test, Router ID 4.4.4.4:
 Total number of peers : 2                 Peers in established state : 2

  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State  Pr
efRcv
  10.80.224.34    4           5       33       40     0 00:29:35 Established    
    2
  10.180.25.1     4          10       42       38     0 00:29:51 Established    
    2

    0
```
Ya se tienen formada las adyacencias en los equipos, se procede a probar comunicación en ellos
```html
<Core-3>ping -vpn-instance Test 10.180.25.2
  PING 10.180.25.2: 56  data bytes, press CTRL_C to break
    Reply from 10.180.25.2: bytes=56 Sequence=1 ttl=255 time=16 ms
    Reply from 10.180.25.2: bytes=56 Sequence=2 ttl=255 time=4 ms
    Reply from 10.180.25.2: bytes=56 Sequence=3 ttl=255 time=3 ms
    Reply from 10.180.25.2: bytes=56 Sequence=4 ttl=255 time=3 ms
    Reply from 10.180.25.2: bytes=56 Sequence=5 ttl=255 time=4 ms

  --- 10.180.25.2 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 3/6/16 ms
 
<Core-3>ping -vpn-instance Test 10.180.25.15
  PING 10.180.25.15: 56  data bytes, press CTRL_C to break
    Reply from 10.180.25.15: bytes=56 Sequence=1 ttl=255 time=1 ms
    Reply from 10.180.25.15: bytes=56 Sequence=2 ttl=255 time=2 ms
    Reply from 10.180.25.15: bytes=56 Sequence=3 ttl=255 time=1 ms
    Reply from 10.180.25.15: bytes=56 Sequence=4 ttl=255 time=2 ms
    Reply from 10.180.25.15: bytes=56 Sequence=5 ttl=255 time=2 ms

  --- 10.180.25.15 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 1/1/2 ms
 
<Core-3>ping -vpn-instance Test 10.180.17.7
  PING 10.180.17.7: 56  data bytes, press CTRL_C to break
    Reply from 10.180.17.7: bytes=56 Sequence=1 ttl=255 time=1 ms
    Reply from 10.180.17.7: bytes=56 Sequence=2 ttl=255 time=1 ms
    Reply from 10.180.17.7: bytes=56 Sequence=3 ttl=255 time=1 ms
    Reply from 10.180.17.7: bytes=56 Sequence=4 ttl=255 time=1 ms
    Reply from 10.180.17.7: bytes=56 Sequence=5 ttl=255 time=2 ms

  --- 10.180.17.7 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 1/1/2 ms
 
<Core-3>ping -vpn-instance Test 10.180.17.2
  PING 10.180.17.2: 56  data bytes, press CTRL_C to break
    Reply from 10.180.17.2: bytes=56 Sequence=1 ttl=255 time=11 ms
    Reply from 10.180.17.2: bytes=56 Sequence=2 ttl=255 time=3 ms
    Reply from 10.180.17.2: bytes=56 Sequence=3 ttl=255 time=3 ms
    Reply from 10.180.17.2: bytes=56 Sequence=4 ttl=255 time=3 ms
    Reply from 10.180.17.2: bytes=56 Sequence=5 ttl=255 time=3 ms

  --- 10.180.17.2 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 3/4/11 ms

<Core-2>disp bgp vpnv4 vpn Test peer
 
 BGP local router ID : 0.0.0.0
 Local AS number : 5

 VPN-Instance Test, Router ID 4.4.4.4:
 Total number of peers : 2                 Peers in established state : 2

  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State  Pr
efRcv
  10.80.224.34    4           5       33       40     0 00:29:35 Established    
    2
  10.180.25.1     4          10       42       38     0 00:29:51 Established    
    2

    0
```
Se comprueba en los routers de acceso comunicación entre sí
```html
<R1>disp bgp vpnv4 vpn Test routing-table

 BGP Local router ID is 0.0.0.0 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete


 VPN-Instance Test, Router ID 3.3.3.3:

 Total Number of Routes: 8
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   10.80.115.0/27     0.0.0.0         0                     0      ?
   i                     10.80.115.1     0          100        0      ?
 *>   10.80.115.2/32     0.0.0.0         0                     0      ?
 *>   10.80.213.96/28    0.0.0.0         0                     0      i
 *                       0.0.0.0         0                     0      ?
 *>   10.80.213.97/32    0.0.0.0         0                     0      ?
 *>i  10.180.17.0/29     10.80.115.1     0          100        0      ?
 *>i  10.180.25.0/28     10.180.17.1     0          100        0      10?
<R1>
<R1>ping -c 5 -m 10 -vpn Test 169.254.161.1
  PING 169.254.161.1: 56  data bytes, press CTRL_C to break
    Reply from 169.254.161.1: bytes=56 Sequence=1 ttl=250 time=80 ms
    Reply from 169.254.161.1: bytes=56 Sequence=2 ttl=250 time=80 ms
    Reply from 169.254.161.1: bytes=56 Sequence=3 ttl=250 time=80 ms
    Reply from 169.254.161.1: bytes=56 Sequence=4 ttl=250 time=80 ms
    Reply from 169.254.161.1: bytes=56 Sequence=5 ttl=250 time=90 ms

  --- 169.254.161.1 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 80/82/90 ms
```
#### Clientes
Para el cliente de la red 169.254.161.0/30 se configura la ip estática 169.254.161.2, para el cliente con la red 10.80.213.96/28 se configurar direccionamiento dinámico
```html
[R1]dhcp enable 
Info: The operation may take a few seconds. Please wait for a moment.done.
[R1]disp curr int ethe0/0/1
#
interface Ethernet0/0/1
 description to-Client.PC1
 ip binding vpn-instance Test
 ip address 10.80.213.97 255.255.255.240
#
return

[R1]ip pool Client.PC1
Info:It's successful to create an IP address pool.
[R1-ip-pool-Client.PC1]network 10.80.213.96 mask 28
[R1-ip-pool-Client.PC1]gateway-list 10.80.213.97	
[R1-ip-pool-Client.PC1]dns-list 8.8.8.8 8.8.4.4
[R1-ip-pool-Client.PC1]lease day 5 hour 23 minute 59
[R1-ip-pool-Client.PC1]vpn-instance Test
[R1-ip-pool-Client.PC1]excluded-ip-address 10.80.213.98 10.80.213.99
[R1-ip-pool-Client.PC1]disp this
#
ip pool Client.PC1
 vpn-instance Test
 gateway-list 10.80.213.97
 network 10.80.213.96 mask 255.255.255.240
 excluded-ip-address 10.80.213.98 10.80.213.99
 lease day 5 hour 6 minute 59
 dns-list 8.8.8.8 8.8.4.4
#
return
[R1-ip-pool-Client.PC1]int ethe0/0/1
[R1-Ethernet0/0/0]dhcp select global
```
Toca comprobar con el equipo cliente la configuración ejecutando los comandos *'ipconfig /release'* y *'ipconfig /renew'* para Windows en caso de equipos Linux es *'sudo dhclient'* *'ifconfig ó ip address'*
*PC1*
```html
PC>ipconfig

Link local IPv6 address...........: fe80::5689:98ff:fe20:17a0
IPv6 address......................: :: / 128
IPv6 gateway......................: ::
IPv4 address......................: 10.80.213.110
Subnet mask.......................: 255.255.255.240
Gateway...........................: 10.80.213.97
Physical address..................: 54-89-98-20-17-A0
DNS server........................: 8.8.8.8
                                    8.8.4.4
PC>ping 169.254.161.2

Ping 169.254.161.2: 32 data bytes, Press Ctrl_C to break
From 169.254.161.2: bytes=32 seq=1 ttl=121 time=109 ms
From 169.254.161.2: bytes=32 seq=2 ttl=121 time=94 ms
From 169.254.161.2: bytes=32 seq=3 ttl=121 time=125 ms
From 169.254.161.2: bytes=32 seq=4 ttl=121 time=109 ms
From 169.254.161.2: bytes=32 seq=5 ttl=121 time=78 ms

--- 169.254.161.2 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 78/103/125 ms
```
*PC2*
```html
PC>ipconfig

Link local IPv6 address...........: fe80::5689:98ff:fed5:5cb4
IPv6 address......................: :: / 128
IPv6 gateway......................: ::
IPv4 address......................: 169.254.161.2
Subnet mask.......................: 255.255.255.252
Gateway...........................: 169.254.161.1
Physical address..................: 54-89-98-D5-5C-B4
DNS server........................:

PC>ping 10.80.213.110

Ping 10.80.213.110: 32 data bytes, Press Ctrl_C to break
From 10.80.213.110: bytes=32 seq=1 ttl=121 time=125 ms
From 10.80.213.110: bytes=32 seq=2 ttl=121 time=94 ms
From 10.80.213.110: bytes=32 seq=3 ttl=121 time=125 ms
From 10.80.213.110: bytes=32 seq=4 ttl=121 time=125 ms
From 10.80.213.110: bytes=32 seq=5 ttl=121 time=109 ms

--- 10.80.213.110 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 94/115/125 ms
```
En R1 se comprueba las configuraciones realizadas y se ve un host activo que es la PC1
```html
[R1]disp curr int ethe0/0/1
#
interface Ethernet0/0/1
 description to-Client.PC1
 ip binding vpn-instance Test
 ip address 10.80.213.97 255.255.255.240
 dhcp select global
#
return
<R1>disp ip pool vpn Test 
  --------------------------------------------------------------------------
  Pool-name                   Position       Gateway              Mask
  --------------------------------------------------------------------------
  Client.PC1                  Local        10.80.213.97     255.255.255.240 
  --------------------------------------------------------------------------


<R1>disp ip pool name Client.PC1 
  Pool-name      : Client.PC1
  Pool-No        : 0
  Lease          : 5 Days 6 Hours 59 Minutes
  Domain-name    : -
  DNS-server0    : 8.8.8.8         
  DNS-server1    : 8.8.4.4         
  NBNS-server0   : -               
  Netbios-type   : -               
  Position       : Local           Status           : Unlocked
  Gateway-0      : 10.80.213.97    
  Mask           : 255.255.255.240
  VPN instance   : Test
 -----------------------------------------------------------------------------
         Start           End     Total  Used  Idle(Expired)  Conflict  Disable
 -----------------------------------------------------------------------------
    10.80.213.97   10.80.213.110    13     1         10(0)         0        2
 -----------------------------------------------------------------------------
```
#### Tunnel GRE

![w2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEicbjhs1N3PEaWBQpAN7S4qjlrp9c891vXbZLfbb_zJc_b_5VFBJ616cO8xu5XnhNGeZ77fztvgtUb-CZCrKd3q9RNlRtDS0TuiyhSmY0U0tTQJk6oA9sgAvYXinyhlQNC_tX9zsIXIqZftw_3Svxd0gQ9Sepbjit8ZZGyLZZrDqWSOjHITbcdH0XnUxQ/s1520/Diagrama%20en%20blanco.jpeg)
Antes de empezar a configurar el túnel se verifica los saltos que dan los equipos para poder comunicarse, se comprueba en los equipos de acceso
```html
PC>tracert 169.254.161.2

traceroute to 169.254.161.2, 8 hops max
(ICMP), press Ctrl+C to stop
 1  10.80.213.97   16 ms  31 ms  16 ms
 2  10.80.115.1   46 ms  47 ms  47 ms
 3  10.80.115.2   31 ms  47 ms  47 ms
 4  10.180.25.2   47 ms  63 ms  46 ms
 5  10.180.17.1   63 ms  62 ms  63 ms
 6  10.180.25.2   62 ms  79 ms  62 ms
 7  10.80.224.34   78 ms  78 ms  78 ms
 8  169.254.161.2   110 ms  109 ms  125 ms

PC>tracert 10.80.213.110

traceroute to 10.80.213.110, 8 hops max
(ICMP), press Ctrl+C to stop
 1  169.254.161.1   16 ms  16 ms  16 ms
 2  10.80.224.33   47 ms  31 ms  47 ms
 3  10.80.224.34   47 ms  47 ms  46 ms
 4  10.180.17.2   63 ms  62 ms  94 ms
 5  10.180.25.1   63 ms  62 ms  63 ms
 6  10.180.17.2   109 ms  109 ms  94 ms
 7  10.80.115.2   94 ms  93 ms  110 ms
 8  10.80.213.110   140 ms  125 ms  125 ms

<R2>tracert -vpn Test 10.80.213.97

 traceroute to Test 10.80.213.97(10.80.213.9
7), max hops: 30 ,packet length: 40,press CTRL_C to break 

 1 10.80.224.33 50 ms  50 ms  50 ms 

 2 10.80.224.34 30 ms 10.180.25.1 40 ms 10.80.224.34 50 ms 

 3 10.180.17.2 50 ms 10.80.224.33 60 ms 10.180.17.2 50 ms 

 4 10.180.25.1 60 ms  60 ms  50 ms 

 5 10.180.17.2 60 ms  60 ms  50 ms 

 6 10.80.115.2 60 ms  100 ms  60 ms 
```
El túnel usará la misma instancia VPN creada en los equipos
```html
[R1]int tunnel 0/0/0
[R1-Tunnel0/0/0]
[R1-Tunnel0/0/0]ip binding vpn-instance Test
[R1-Tunnel0/0/0]ip add 10.10.10.1 30
[R1-Tunnel0/0/0]desc to-R2.Tunnel0/0/0
[R1-Tunnel0/0/0]tunnel-protocol gre
[R1-Tunnel0/0/0]source 10.80.115.2
[R1-Tunnel0/0/0]destination vpn-instance Test 10.80.224.34
Sep  2 2022 16:23:20-08:00 R1 %%01IFNET/4/LINK_STATE(l)[12]:The line protocol IP
 on the interface Tunnel0/0/0 has entered the UP state.
[R1-Tunnel0/0/0]disp this
#
interface Tunnel0/0/0
 description to-R2.Tunnel0/0/0
 ip binding vpn-instance Test
 ip address 10.10.10.1 255.255.255.252
 tunnel-protocol gre
 source 10.80.115.2
 destination vpn-instance Test 10.80.224.34
#
return
```
El protocolo de encapsulación es *GRE* y se especifica una dirección de origen y destino para el túnel, cabe mencionar que es necesario que estos dos equipos tengan comunicación con las ip declaradas en la configuración. El direccionamiento para el túnel debe ser diferente al que se tiene en el esto de la red como se ve en la imagen.

Se hace la configuración correspondiente para *R2*
```html
[R2]int tunnel 0/0/0
[R2-Tunnel0/0/0]ip binding vpn-instance Test
[R2-Tunnel0/0/0]ip add 10.10.10.2 30
[R2-Tunnel0/0/0]desc to-R1.Tunnel0/0/0
[R2-Tunnel0/0/0]tunnel-protocol gre
[R2-Tunnel0/0/0]source 10.80.224.34
[R2-Tunnel0/0/0]destination vpn-instance Test 10.80.115.2
Sep  2 2022 16:27:16-08:00 R2 %%01IFNET/4/LINK_STATE(l)[1]:The line protocol IP 
on the interface Tunnel0/0/0 has entered the UP state.
[R2-Tunnel0/0/0]disp this
#
interface Tunnel0/0/0
 description to-R1.Tunnel0/0/0
 ip binding vpn-instance Test
 ip address 10.10.10.2 255.255.255.252
 tunnel-protocol gre
 source 10.80.224.34
 destination vpn-instance Test 10.80.115.2
#
return
```
Comprobamos las configuraciones hechas previamente
```html
<R2>ping -vpn Test 10.10.10.1
  PING 10.10.10.1: 56  data bytes, press CTRL_C to break
    Reply from 10.10.10.1: bytes=56 Sequence=1 ttl=255 time=100 ms
    Reply from 10.10.10.1: bytes=56 Sequence=2 ttl=255 time=60 ms
    Reply from 10.10.10.1: bytes=56 Sequence=3 ttl=255 time=60 ms
    Reply from 10.10.10.1: bytes=56 Sequence=4 ttl=255 time=90 ms
    Reply from 10.10.10.1: bytes=56 Sequence=5 ttl=255 time=60 ms

  --- 10.10.10.1 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 60/74/100 ms

<R2>tracert -vpn Test 10.10.10.1

 traceroute to Test 10.10.10.1(10.10.10.1), max hops: 30 ,packet length: 40,pres
s CTRL_C to break 

 1 10.10.10.1 80 ms  70 ms  100 ms
```
Con el comando *'tracert'* podemos observar que hay comunicación pero además vemos que llega al equipo dando un "único salto" y no lo hace pasando por cada capa, (agregación, núcleo y acceso) el paquete llega directo al nodo como si fuera un enlace punto a punto, ahora veamos la información que dal equipo sobre el nodo
```html
[R1]disp tunnel-info all
 * -> Allocated VC Token
Tunnel ID           Type                 Destination           Token
----------------------------------------------------------------------
0x1                 gre                   10.80.224.34           1 
```
* *Tunnel ID:* Es la notación que da el sistema al túnel en formato hexadecimal
* *Token:* Identificación del túnel y lo asigna el sistema

> En caso de tener más túneles en el nodo, podemos filtrar de mejor manera la búsqueda con el id que le asigne el nodo *'display tunnel-info tunnel-id 1'* o bien viendolo como una interfaz *'display interface Tunnel 0/0/0'*

Hechas estas configuraciones aún los equipos clientes (PC1 y PC2) siguen dando todos los saltos sobre la red, para que ya no ocurra esto agregamos una ruta estática a los nodos y ponemos que el next hop será por el túnel o bien podemos poner la ip del túnel según corresponda
```html
[R1]ip route-static vpn-instance Test 169.254.161.0 30 Tunnel 0/0/0

[R2]ip route-static vpn-instance Test 10.80.213.96 28 Tunnel 0/0/0

<R1>tracert -vpn Test 169.254.161.2

 traceroute to Test 169.254.161.2(169.254.1
61.2), max hops: 30 ,packet length: 40,press CTRL_C to break 

 1 10.10.10.2 60 ms  80 ms  100 ms 

 2 169.254.161.2 110 ms  90 ms  110 ms

<R2>ping -a 10.10.10.2 -vpn Test 10.80.213.110
  PING 10.80.213.110: 56  data bytes, press CTRL_C to break
    Reply from 10.80.213.110: bytes=56 Sequence=1 ttl=127 time=100 ms
    Reply from 10.80.213.110: bytes=56 Sequence=2 ttl=127 time=70 ms
    Reply from 10.80.213.110: bytes=56 Sequence=3 ttl=127 time=120 ms
    Reply from 10.80.213.110: bytes=56 Sequence=4 ttl=127 time=110 ms
    Reply from 10.80.213.110: bytes=56 Sequence=5 ttl=127 time=140 ms

  --- 10.80.213.110 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 70/108/140 ms
```
Y ahora sí ya vemos que la comunicación es directa con la red, se comprueba en los host
```html
PC>ipconfig

Link local IPv6 address...........: fe80::5689:98ff:fe20:17a0
IPv6 address......................: :: / 128
IPv6 gateway......................: ::
IPv4 address......................: 10.80.213.110
Subnet mask.......................: 255.255.255.240
Gateway...........................: 10.80.213.97
Physical address..................: 54-89-98-20-17-A0
DNS server........................: 8.8.8.8
                                    8.8.4.4
PC>tracert 169.254.161.2

traceroute to 169.254.161.2, 8 hops max
(ICMP), press Ctrl+C to stop
 1  10.80.213.97   <1 ms  31 ms  15 ms
 2  10.10.10.2   79 ms  78 ms  93 ms
 3  169.254.161.2   110 ms  109 ms  94 ms

PC>ipconfig

Link local IPv6 address...........: fe80::5689:98ff:fed5:5cb4
IPv6 address......................: :: / 128
IPv6 gateway......................: ::
IPv4 address......................: 169.254.161.2
Subnet mask.......................: 255.255.255.252
Gateway...........................: 169.254.161.1
Physical address..................: 54-89-98-D5-5C-B4
DNS server........................:

PC>tracert 10.80.213.110

traceroute to 10.80.213.110, 8 hops max
(ICMP), press Ctrl+C to stop
 1  169.254.161.1   <1 ms  32 ms  15 ms
 2  10.10.10.1   94 ms  78 ms  94 ms
 3  10.80.213.110   109 ms  110 ms  109 ms
```
## [Atrás](./)