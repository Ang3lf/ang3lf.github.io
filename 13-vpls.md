---
title: VPLS - VSI
layout: default
---
*Virtual Private LAN Service* Es un Tipo de VPN de capa 2 punto a multipunto que permite conectar redes LAN de distintos puntos entre sí utilizando una red IP/MPLS, a este tipo de servicio se le conoce como *_L2VPN_* existe otro tipo que es conocido como *_L3VPN_* que a diferencia de L2VPN este usa protocolos de enrutamiento para la comunicación usando instancias de VPN dentro del protocolo y anuncia las redes utilizando MPLS en las interfaces, una característica importante de este tipo de MPLS es que se le puede configurar algún tipo de QoS para que el tráfico de MPLS (Labels) sea mejor conforme a lo necesitemos.

Existen diferentes tipos de implementación de VPLS, en este caso será la configuración típica de crear una red punto a punto entre 2 redes distintas, estremos trabajando con la siguiente topología:

![x1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjbj8-CRVXN3hxhY_xCqYoXhQDj6cax32Pkqga0ChUQrQHqdRcVbrYvtV8Vc3st843zTNR7errYzFvZMc05cbXPgA0ZWH0PQ7Rb9TnbM1vLn2QhG5078PZ5bfL6wSW8kj70RxMPe9l6ulny9aoP4JPoupOisS0zhey4tuYWJcRuuXMLcylklq8i5r4esw/s1800/Diagrama%20en%20blanco.png)

Primero para realizar la configuración de *L2VPN* debemos tener routing entre los equipos llegando a las interfaces y loopbacks de cada nodo, en este ejemplo se configuro el protocolo de enrutamiento OSPF, se implementaron equipos Huawei y Nokia para las capas de Acceso, Agregación y Núcleo, para los equipos de los clientes (Customer...) hay equipos Huawei y Cisco.

### Configuración

![x2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiz-w12URYFLuqWJa_UIestWg4DxtxmueKxQd904fqdWndOgpxtF-c1ie2dyb4DzenjvUFr8ahGGkxVOksJsna8VhtvUA4MbLMUWL0RNVhT83DXleoNQ8MTIMidLiRuReefTsKNIh8T8o1IVyYY6mTnyPdMAh9Y2mLDoWSBceeBurDSbeFnnF-YI8kZdQ/s1040/Diagrama%20en%20blanco%20(2).png)

Comenzamos configurando OSPF en los equipos Nokia:

```html
*A:Aggre-3>config>router# ospf
*A:Aggre-3>config>router>ospf$ area 0
*A:Aggre-3>config>router>ospf>area$ interface "system"
*A:Aggre-3>config>router>ospf>area>if$ exit
*A:Aggre-3>config>router>ospf>area$ interface "to-Aggre-
"to-Aggre-1.Ethernet1/0/2"              "to-Aggre-2.Ethernet1/0/2"
"to-Aggre-4.Port1/1/4"
*A:Aggre-3>config>router>ospf>area$ interface "to-Aggre-4.Port1/1/4"
*A:Aggre-3>config>router>ospf>area>if$ interface-type point-to-point
*A:Aggre-3>config>router>ospf>area>if$ exit
*A:Aggre-3>config>router>ospf>area$ interface "to-Aggre-2.Ethernet1/0/2"
*A:Aggre-3>config>router>ospf>area>if$ interface-type point-to-point
*A:Aggre-3>config>router>ospf>area>if$ back
*A:Aggre-3>config>router>ospf>area$ interface "to-Aggre-1.Ethernet1/0/2" interface-type point-to-point
*A:Aggre-3>config>router>ospf>area$ exit
*A:Aggre-3>config>router>ospf$ info
----------------------------------------------
            area 0.0.0.0 router-id 1.1.1.1
                interface "system"
                    no shutdown
                exit
                interface "to-Aggre-1.Ethernet1/0/2"
                    interface-type point-to-point
                    no shutdown
                exit
                interface "to-Aggre-2.Ethernet1/0/2"
                    interface-type point-to-point
                    no shutdown
                exit
                interface "to-Aggre-4.Port1/1/4"
                    interface-type point-to-point
                    no shutdown
                exit
            exit
```
```html
*A:Aggre-4>config>router>ospf>$ info
----------------------------------------------
            area 0.0.0.0 router-id 2.2.2.2
                interface "system"
                    no shutdown
                exit
                interface "to-Aggre-1.Ethernet1/0/4"
                    interface-type point-to-point
                    no shutdown
                exit
                interface "to-Aggre-2.Ethernet1/0/3"
                    interface-type point-to-point
                    no shutdown
                exit
                interface "to-Aggre-3.Port1/1/2"
                    interface-type point-to-point
                    no shutdown
                exit
```
> Al configurar 'interface-type point-to-point' en equipos Nokia lo debemos tener en cuenta si vamos a configurar OSPF con otro proveedor ya que no van a tener los mismo 'mtu' en las interfaces y esto va a impedir que haya adyacencias OSPF correctas, recordemos que hay varios *network-type* en OSPF: P2P, P2MP, Non broadcast y broadcast por lo regular se encuentran con Broadcast al configurarlo por primera vez

Se configura OSPF área 0 en los equipos Huawei:
```html
[~Aggre-1]ospf 1 router-id 12.12.12.12
[*Aggre-1-ospf-1]area 0
[*Aggre-1-ospf-1-area-0.0.0.0]network 10.90.214.16 0.0.0.15
[*Aggre-1-ospf-1-area-0.0.0.0]network 10.90.214.48 0.0.0.15
[*Aggre-1-ospf-1-area-0.0.0.0]net 10.110.214.24 0.0.0.7
[*Aggre-1-ospf-1-area-0.0.0.0]net 10.110.214.0 0.0.0.7
[*Aggre-1-ospf-1-area-0.0.0.0]commit
[~Aggre-1-ospf-1-area-0.0.0.0]disp this
 area 0.0.0.0
  network 10.90.214.16 0.0.0.15
  network 10.90.214.48 0.0.0.15
  network 10.110.214.0 0.0.0.7
  network 10.110.214.24 0.0.0.7
#
return
```
```html
[~Aggre-2-ospf-1]disp this
#
ospf 1 router-id 13.13.13.13
 area 0.0.0.0
  network 10.90.214.32 0.0.0.15
  network 10.90.214.64 0.0.0.15
  network 10.110.214.8 0.0.0.7
  network 10.110.214.64 0.0.0.7
#
return
```
```html
[~Core-1-ospf-1]disp this
#
ospf 1 router-id 14.14.14.14
 area 0.0.0.0
  network 10.90.214.0 0.0.0.15
  network 10.90.214.16 0.0.0.15
#
return
[~Core-2-ospf-1]disp this
#
ospf 1 router-id 15.15.15.15
 area 0.0.0.0
  network 10.90.214.0 0.0.0.15
  network 10.90.214.32 0.0.0.15
#
return
[~Access-1-ospf-1]disp this
#
ospf 1 router-id 16.16.16.16
 area 0.0.0.0
  network 10.90.214.48 0.0.0.15
  network 10.240.0.0 0.0.0.31
#
return
[~Access-2-ospf-1]disp this
#
ospf 1 router-id 17.17.17.17
 area 0.0.0.0
  network 10.1.1.0 0.0.0.3
  network 10.90.214.64 0.0.0.15
#
return
```
La configuración para los CPE´s de los clientes es (ambos equipos están dentro del área 0, también como medida de seguridad se puede configurar una ruta estática en estos para que personal no autorizado no vea la tabla de rutas):
```html
<Site1>disp ip int brief
*down: administratively down
^down: standby
(l): loopback
(s): spoofing
(E): E-Trunk down
The number of interface that is UP in Physical is 9
The number of interface that is DOWN in Physical is 0
The number of interface that is UP in Protocol is 4
The number of interface that is DOWN in Protocol is 5

Interface                         IP Address/Mask      Physical   Protocol
GigabitEthernet0/0/0              10.240.0.2/27        up         up
GigabitEthernet0/0/1              192.168.168.1/24     up         up
GigabitEthernet0/0/2              unassigned           up         down
GigabitEthernet0/0/3              unassigned           up         down
GigabitEthernet0/0/4              unassigned           up         down
GigabitEthernet0/0/5              unassigned           up         down
GigabitEthernet0/0/6              unassigned           up         down
LoopBack1                         18.18.18.18/32       up         up(s)
NULL0                             unassigned           up         up(s)

<Site1>disp curr | section ospf
#
ospf 1 router-id 18.18.18.18
 silent-interface GigabitEthernet0/0/1
 area 0.0.0.0
  network 10.240.0.0 0.0.0.31
  network 192.168.168.0 0.0.0.255
#
return

Site2(config)#router ospf 1
Site2(config-router)#router-id 19.19.19.19
Site2(config-router)#network 10.1.1.0 0.0.0.3 area 0
Site2(config-router)#network 172.255.240.0 0.0.0.255 area 0
Site2(config-router)#passive-interface ethernet 0/1
Site2(config-router)#do show runn | section ospf 1
Site2#show runni | section ospf
 description ospf1
router ospf 1
 router-id 19.19.19.19
 passive-interface Ethernet0/1
 network 10.1.1.0 0.0.0.3 area 0
 network 19.19.19.19 0.0.0.0 area 0
 network 172.255.240.0 0.0.0.255 area 0
!
```
Para que los equipos hagan adyacencia OSPF se debe verificar que los mtu de las interfaces sean los mismos y el tipo de *network-type* esto es porque los proveedores de los equipos tienen diferentes valores en las interfaces, para ello debemos configurar en las interfaces de los equipos Huawei (Aggre-1, Aggre-2) que van conectados a los equipos Nokia (Aggre-3, Aggre-4) con el tipo de network en *P2P* ya que en los nodos de Nokia se configuró así

```html
[~Aggre-1-Ethernet1/0/2]ospf enable 1 area 0.0.0.0
[~Aggre-1-Ethernet1/0/2]ospf network-type p2p
[~Aggre-1-Ethernet1/0/2]mtu 1500
<Aggre-1>reset ospf 1 process
[~Aggre-1-Ethernet1/0/3]disp this
#
interface Ethernet1/0/3
 description to-Aggre-4.Port1/1/2
 undo shutdown
 ip address 10.110.214.25 255.255.255.248
 ospf network-type p2p
 ospf enable 1 area 0.0.0.0
 dcn mode vlan
#
return
[~Aggre-2-Ethernet1/0/2]disp this
#
interface Ethernet1/0/2
 description to-Aggre-3.Port1/1/2
 undo shutdown
 ip address 10.110.214.9 255.255.255.248
 ospf network-type p2p
 ospf enable 1 area 0.0.0.0
 dcn mode vlan
#
return

[~Aggre-2-Ethernet1/0/3]disp this
#
interface Ethernet1/0/3
 description to-Aggre-4.Port1/1/3
 undo shutdown
 ip address 10.110.214.33 255.255.255.248
 ospf network-type p2p
 ospf enable 1 area 0.0.0.0
 dcn mode vlan
#
return
<Aggre-2>reset ospf 1 process
```
> Con 'reset ospf 1 process' hacemos que el número de proceso indicado OSPF en el router reinicie y empiece a buscar vecinos de este protocolo nuevamente

Ahora verificamos que se hayan hecho las adyacencias en los nodos:
```html
A:Aggre-3>config>router>ospf# show router ospf neighbor

===============================================================================
OSPFv2 (0) all neighbors
===============================================================================
Interface-Name                   Rtr Id          State      Pri  RetxQ   TTL
   Area-Id
-------------------------------------------------------------------------------
to-Aggre-1.Ethernet1/0/2         12.12.12.12     Full       1    0       33
   0.0.0.0
to-Aggre-2.Ethernet1/0/2         13.13.13.13     Full       1    0       33
   0.0.0.0
to-Aggre-4.Port1/1/4             2.2.2.2         Full       1    0       33
   0.0.0.0
-------------------------------------------------------------------------------
No. of Neighbors: 3
===============================================================================

A:Aggre-4>config>router>ospf# show router ospf neighbor

===============================================================================
OSPFv2 (0) all neighbors
===============================================================================
Interface-Name                   Rtr Id          State      Pri  RetxQ   TTL
   Area-Id
-------------------------------------------------------------------------------
to-Aggre-1.Ethernet1/0/4         12.12.12.12     Full       1    0       35
   0.0.0.0
to-Aggre-2.Ethernet1/0/3         13.13.13.13     Full       1    0       32
   0.0.0.0
to-Aggre-3.Port1/1/2             1.1.1.1         Full       1    0       39
   0.0.0.0
-------------------------------------------------------------------------------
No. of Neighbors: 3
===============================================================================
<Aggre-1>disp ospf 1 peer

(M) Indicates MADJ neighbor


          OSPF Process 1 with Router ID 12.12.12.12
                Neighbors

 Area 0.0.0.0 interface 10.90.214.18 (Eth1/0/1)'s neighbors
 Router ID: 14.14.14.14          Address: 10.90.214.17
   State: Full           Mode:Nbr is Master     Priority: 1
   DR: 10.90.214.18      BDR: 10.90.214.17      MTU: 0
   Dead timer due in  38  sec
   Retrans timer interval: 5
   Neighbor is up for 00h01m36s
   Neighbor Up Time : 2022-09-24 23:57:24
   Authentication Sequence: [ 0 ]

 Area 0.0.0.0 interface 10.110.214.1 (Eth1/0/2)'s neighbors
 Router ID: 1.1.1.1              Address: 10.110.214.2
   State: Full           Mode:Nbr is Slave      Priority: 1
   DR: None              BDR: None              MTU: 8922
   Dead timer due in  31  sec
   Retrans timer interval: 5
   Neighbor is up for 00h13m49s
   Neighbor Up Time : 2022-09-24 23:45:12
   Authentication Sequence: [ 0 ]

 Area 0.0.0.0 interface 10.110.214.25 (Eth1/0/3)'s neighbors
 Router ID: 2.2.2.2              Address: 10.110.214.26
   State: Full           Mode:Nbr is Slave      Priority: 1
   DR: None              BDR: None              MTU: 8922
   Dead timer due in  39  sec
   Retrans timer interval: 5
   Neighbor is up for 00h13m41s
   Neighbor Up Time : 2022-09-24 23:45:21
   Authentication Sequence: [ 0 ]

<Aggre-2>disp ospf peer

(M) Indicates MADJ neighbor


          OSPF Process 1 with Router ID 13.13.13.13
                Neighbors

 Area 0.0.0.0 interface 10.90.214.34 (Eth1/0/1)'s neighbors
 Router ID: 15.15.15.15          Address: 10.90.214.33
   State: Full           Mode:Nbr is Master     Priority: 1
   DR: 10.90.214.34      BDR: 10.90.214.33      MTU: 0
   Dead timer due in  31  sec
   Retrans timer interval: 5
   Neighbor is up for 00h00m48s
   Neighbor Up Time : 2022-09-24 23:59:18
   Authentication Sequence: [ 0 ]

 Area 0.0.0.0 interface 10.110.214.9 (Eth1/0/2)'s neighbors
 Router ID: 1.1.1.1              Address: 10.110.214.10
   State: Full           Mode:Nbr is Slave      Priority: 1
   DR: None              BDR: None              MTU: 8922
   Dead timer due in  32  sec
   Retrans timer interval: 5
   Neighbor is up for 00h12m12s
   Neighbor Up Time : 2022-09-24 23:47:55
   Authentication Sequence: [ 0 ]

 Area 0.0.0.0 interface 10.110.214.33 (Eth1/0/3)'s neighbors
 Router ID: 2.2.2.2              Address: 10.110.214.34
   State: Full           Mode:Nbr is Slave      Priority: 1
   DR: None              BDR: None              MTU: 8922
   Dead timer due in  30  sec
   Retrans timer interval: 5
   Neighbor is up for 00h12m12s
   Neighbor Up Time : 2022-09-24 23:47:55
   Authentication Sequence: [ 0 ]

<Site1>disp ospf peer brief

         OSPF Process 1 with Router ID 18.18.18.18
                  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State
 0.0.0.0          GigabitEthernet0/0/0             16.16.16.16      Full
 ----------------------------------------------------------------------------
```
#### VPLS
Antes de empezar con la configuración es necesario que comprobemos si tenemos comunicación entre los equipos de Acceso que es donde empezaremos configurando *L2VPN*, debemos comprobar que haya ping a las IP de las loopbacks.

![x3](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj8L3GjovZ-OftSATD5dTVWgblontDeVPYvVKJbvx51GAsAUaJVXpBPSpJiIxN1SL6Jyw9sm15-F45ucbLtTeRgavychOExX2xVLH1YE09bJhIxlIBtYq9caJeKIUWP9G8Obtww5dIoJ-UYCW2sA8B0cmy51tcMy4jP7FnxsdLmGJANcT2cjZDKAkWhpQ/s1800/Diagrama%20en%20blanco%20(1).png) 

```html
<Access-1>disp curr int loo 5
#
interface LoopBack5
 description tesvsipeer
 ip address 50.50.50.50 255.255.255.255
#
return
<Access-1>ping -s 1024 -c 1 51.51.51.51
  PING 51.51.51.51: 1024  data bytes, press CTRL_C to break
    Reply from 51.51.51.51: bytes=1024 Sequence=1 ttl=252 time=27 ms

  --- 51.51.51.51 ping statistics ---
    1 packet(s) transmitted
    1 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 27/27/27 ms
<Access-2>disp curr int loo 5
#
interface LoopBack5
 description testvsimpls
 ip address 51.51.51.51 255.255.255.255
#
return

<Access-2>ping -s 1024 -c 1 50.50.50.50
  PING 50.50.50.50: 1024  data bytes, press CTRL_C to break
    Reply from 50.50.50.50: bytes=1024 Sequence=1 ttl=252 time=25 ms

  --- 50.50.50.50 ping statistics ---
    1 packet(s) transmitted
    1 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 25/25/25 ms    
```
Confirmando la comunicación ya podemos empezar a configurar MPLS en los nodos y habilitarlo en las interfaces para el envío de datos
```html
[~Access-1]mpls
[~Access-1-mpls]quit
[~Access-1]mpls ldp
[~Access-1-mpls-ldp]mpls l2vpn
[~Access-1]mpls lsr-id 50.50.50.50
[*Access-1]mpls ldp remote-peer 51.51.51.51
[*Access-1-mpls-ldp-remote-51.51.51.51]remote-ip 51.51.51.51
[*Access-1-mpls-ldp-remote-51.51.51.51]quit
[*Access-1]int ethe1/0/0
[*Access-1-Ethernet1/0/0]mpls
[*Access-1-Ethernet1/0/0]mpls ldp
[*Access-1-Ethernet1/0/0]commit
[~Access-1-Ethernet1/0/0]disp this
interface Ethernet1/0/0
 description to-Aggre-1.Ethernet1/0/0
 undo shutdown
 ip address 10.90.214.50 255.255.255.240
 mpls
 mpls ldp
 dcn
 dcn mode vlan
#

[*Access-2]mpls
[*Access-2-mpls]mpls ldp
[*Access-2-mpls-ldp]mpls l2vpn
[*Access-2]mpls lsr-id 53.53.53.53
[*Access-2]mpls ldp remote-peer 52.52.52.52
[*Access-2-mpls-ldp-remote-52.52.52.52]remote-ip 52.52.52.52
[*Access-2]int ethe1/0/0
[*Access-2-Ethernet1/0/0]mpls
[*Access-2-Ethernet1/0/0]mpls ldp
[*Access-2-Ethernet1/0/0]commit
[~Access-2-Ethernet1/0/0]disp this
interface Ethernet1/0/0
 description to-Aggre-2.Ethernet1/0/0
 undo shutdown
 ip address 10.90.214.66 255.255.255.240
 mpls
 mpls ldp
 dcn
 dcn mode vlan
#
```
Y tocaría hacer algunas de estás configuraciones en los demás equipos, habilitar MPLS, MPLS LDP y MPLS L2VPN y configurarlo para las interfaces que van a estar dentro de este proceso según vaya correspondiendo, ya que en algunos casos, solo sería para subinterfaces o SAP en Nokia.

Para equipos Huawei podemos hacer *L2VPN* haciendo una *VSI* (Virtual Switch Instance) en los equipos de Acceso, estas VSI trabajan junto con VPLS ya que proporciona servicios VPLS de forma independiente y reenvía paquetes de capa 2 en función de direcciones MAC y etiquetas VLAN
```html
[~Access-1]vsi Test static
[*Access-1-vsi-Test]pwsignal ldp
[*Access-1-vsi-Test-ldp]vsi-id 100
[*Access-1-vsi-Test-ldp]peer 51.51.51.51
[~Access-1]int ethe1/0/0.100
[*Access-1-Ethernet1/0/0.100]vlan-type dot1q 100
[*Access-1-Ethernet1/0/0.100]l2 binding vsi Test
<Access-1>disp curr | begin mpls l2vpn
mpls l2vpn
#
vsi Test static
 pwsignal ldp
  vsi-id 100
  peer 51.51.51.51
  peer 52.52.52.52
#
mpls ldp
 #
 ipv4-family
#
mpls ldp remote-peer 51.51.51.51
 remote-ip 51.51.51.51
```
Se crea la VSI, hacemos que el tipo de transporte sea por LDP para MPLS, el *vsi-id* se configura para que sea 100 esta configuración es muy importante ya que para los otros equipos deben tener el mismo vsi-id para saber manejar el tráfrico, por último creamos una subinterfaz en la que está conectada al otro nodo y hacemos que se haga "unión" a la vsi que acabamos de configurar, se hace lo mismo para el otro Acceso y modificamos valores como los peer
```html
[~Access-2-Ethernet1/0/1.100]disp curr config vsi Test
#
vsi Test static
 pwsignal ldp
  vsi-id 100
  peer 50.50.50.50
#
return
```
Si probamos el siguiente comando veremos que aun el estado de la VSI está down y es porque si bien los nodos pueden comunicarse cada uno respectivamente a los peers los otros equipos no saben a donde deben enviarlo, entonces debemos crear en los otros nodos la subinterfaz 100 con el número de interfaz física dentro del proceso de VPLS para que podamos ver arriba la VSI.
```html
[~Access-1-Ethernet1/0/2.100]disp vsi name Test
--------------------------------------------------------------------------
Vsi                             Mem    PW    Mac       Encap     Mtu   Vsi
Name                            Disc   Type  Learn     Type      Value State
--------------------------------------------------------------------------
Test                            static ldp   unqualify vlan      1500  down
```
Cuando hayamos hecho las configuraciones restantes en los otros nodos y volvemos a comprobar la VSI:
```html
[~Access-1-Ethernet1/0/2.100]disp vsi name Test
--------------------------------------------------------------------------
Vsi                             Mem    PW    Mac       Encap     Mtu   Vsi
Name                            Disc   Type  Learn     Type      Value State
--------------------------------------------------------------------------
Test                            static ldp   unqualify vlan      1500  up

[~Access-2-Ethernet1/0/1.100]disp vsi name Test
--------------------------------------------------------------------------
Vsi                             Mem    PW    Mac       Encap     Mtu   Vsi
Name                            Disc   Type  Learn     Type      Value State
--------------------------------------------------------------------------
Test                            static ldp   unqualify vlan      1500  up
```
Si mostramos el estado de nuestro peer lo veremos que esta activo, en este punto ya podremos hacer pruebas de comunicación o mostrar las direcciones MAC que ha aprendido cada nodo uno del otro.

Podemos ver las direcciones MAC que han aprendido utilizando: *'display mac-address dynamic vsi Test'*
```html
[~Access-1-Ethernet1/0/2.100]disp vsi name Test peer-info

VSI Name: Test                                        Signaling: ldp
--------------------------------------------------------------------
Peer                Transport  Local      Remote      VC
Addr                VC ID      VC Label   VC Label    State
--------------------------------------------------------------------
51.51.51.51         100        48120      48120       up
```
Comprobamos si hay comunicación por medio de la VPLS usando:
```html
<Access-1>ping vpls vsi Test peer 51.51.51.51
  PW PING : FEC 128 PSEUDOWIRE (NEW). Type = vlan, ID = 100 : 100 data bytes, press CTRL_C to break
  Reply from 51.51.51.51: bytes=100 Sequence=1 time=19 ms
  Reply from 51.51.51.51: bytes=100 Sequence=2 time=12 ms
  Reply from 51.51.51.51: bytes=100 Sequence=3 time=12 ms
  Reply from 51.51.51.51: bytes=100 Sequence=4 time=11 ms
  Reply from 51.51.51.51: bytes=100 Sequence=5 time=10 ms

--- FEC : FEC 128 PSEUDOWIRE (NEW). Type = vlan, ID = 100 ping statistics ---
 5 packet(s) transmitted
 5 packet(s) received
 0.00% packet loss
 round-trip min/avg/max = 10/12/19 ms

<Access-1>tracert vpls vsi Test peer 51.51.51.51
  PW Trace Route : FEC 128 PSEUDOWIRE (NEW). Type = vlan, ID = 100, press CTRL_C to break
TTL    Replier            Time    Type      Downstream
0                                 Ingress   10.90.214.49/[48124 1 48120 ]
5      51.51.51.51        36 ms   Egress

<Access-2>ping vpls vsi -c 5 Test peer 50.50.50.50
  PW PING : FEC 128 PSEUDOWIRE (NEW). Type = vlan, ID = 100 : 100 data bytes, press CTRL_C to break
  Reply from 50.50.50.50: bytes=100 Sequence=1 time=12 ms
  Reply from 50.50.50.50: bytes=100 Sequence=2 time=10 ms
  Reply from 50.50.50.50: bytes=100 Sequence=3 time=9 ms
  Reply from 50.50.50.50: bytes=100 Sequence=4 time=27 ms
  Reply from 50.50.50.50: bytes=100 Sequence=5 time=19 ms

--- FEC : FEC 128 PSEUDOWIRE (NEW). Type = vlan, ID = 100 ping statistics ---
 5 packet(s) transmitted
 5 packet(s) received
 0.00% packet loss
 round-trip min/avg/max = 9/15/27 ms
```
Algo a destacar del resultando del comando *'tracert'* es que además de dar un "único salto" muestra en las columnas 'Type' Ingress e Egress y columna 'Dowstream' el valor de la etiqueta que si recordamos cuando ejecutamos 'display vsi name Test peer-info' nos mostraba el valor que tiene el nodo su etiqueta de forma local y al compararlo al ejecutar tracert vemos que la etiqueta es la misma al resultado del comando, con esto podemos ver que manejo de etiquetas hacen los nodos con MPLS, de igual forma podemos ver como es el proceso MPLS capturando tráfico con wireshark o tshark.

Ahora podemos comprobar en los equipos de los clientes Huawei y Cisco si ya tienen comunicación hacia la LAN de cada uno
```html
<Site1>ping 172.255.240.1
  PING 172.255.240.1: 56  data bytes, press CTRL_C to break
    Reply from 172.255.240.1: bytes=56 Sequence=1 ttl=249 time=146 ms
    Reply from 172.255.240.1: bytes=56 Sequence=2 ttl=249 time=31 ms
    Reply from 172.255.240.1: bytes=56 Sequence=3 ttl=249 time=47 ms
    Reply from 172.255.240.1: bytes=56 Sequence=4 ttl=249 time=51 ms
    Reply from 172.255.240.1: bytes=56 Sequence=5 ttl=249 time=69 ms

  --- 172.255.240.1 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 31/68/146 ms

<Site1>ping -c 150 -m 10 -q -s 1024 172.255.240.1
  PING 172.255.240.1: 1024  data bytes, press CTRL_C to break

  --- 172.255.240.1 ping statistics ---
    150 packet(s) transmitted
    150 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 10/35/269 ms

Site2#ping 192.168.168.1 repeat 100 size 1024
Type escape sequence to abort.
Sending 100, 1024-byte ICMP Echos to 192.168.168.1, timeout is 2 seconds:
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Success rate is 100 percent (100/100), round-trip min/avg/max = 12/44/285 ms
```
Ahora comprobaremos en 2 equipos con Windows 10 de ambos sitios (Site1 and Site2) si pueden acceder a recursos compartidos, para ello veamos primero el direccionamiento de cada equipo

##### Site 1
![x4](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgKo7KgEmaEm6WksTGbXIjFDXRHR3pYmcfbQORaLH8ijx_zKLiwQ1g97SAXuhRysblTGRKwI4hz92TiLU5Y2u8oO8f6AErGSmxk4MZkcq8Ysge-aOyarVQd2uDoPdq462T3SGPH2cODcfoJInCWvqvJchHLYIxPs1RouK2gPmEsnIswuqrMW-sf5y2tHA/s693/1.JPG)
##### Site 2
![x5](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiSasYUe_dkUMMAF2qp7enI-_v3k4gfCnJEAXMrbRDQLySjkfBAeB3T5pAvANLJPRn4PcvvlZhRkP_mUtEKHxaj18JD49A-I5j75elled79x7HrkHAVxN1_D6t7j60zCu9oNfgXyjkcnIJ3BLDLkl0A487bOv_j0ekjjdBURcFl0Y_m3OwLZsjeONkphA/s641/2.JPG)
Ambos equipos ya tienen comunicación entonces podríamos compartir un archivo y acceder al mismo

##### Site 1
![x6](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgt1Apg7WeWoqOtnZ4erc7AK5H9HxGFUKvN1Ku_AN5LPLHdXuPoUejnrrEX1pcCjYniYRW86VjnQCzYYfkFxzKmz8od6Z0vZZYsap--0ICjfz5soc1-aCH-qZ8F-CjI56H4fH8sWXJYMLhyejp7RZa_UQWKXfLoV9k2pbczYJ-qSdQTcXOgVLhu_7KIig/s597/1,1.JPG)

##### Site 2
![x7](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgPhQ8RQDoQdRpbaXBd-rzo1dHmfIQm9DlcOWCMZLY8rLZdQVUD4VYSudBivUG0Do8pUIsL8x4VSpACoKUo2gsy4bxLy_fSgb_42cYxDjc4X--xfsXQ97BKUssfwYZIG7RQ70SWwjXz1DIfy2IRDrro2w1hPOIPruUbVwPIU1T3X2ELfPzQWAZiR6hIVw/s505/2,2.JPG)

Site 2 ya tiene accesabilidad al recurso compartido de Site 1, por lo que confirmamos que los equipos tienen conexión punto a punto

##### Site 1
![x8](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjAGQCwvG-xiDeadz1wQZgnfPUHluu5PO2PJ0CH8TFx__9cJVYO0Ht24xWit3Mjh5k3nKTUJgProXtT4MkhoMwHBTgwfMDVxh15LY8VJ5OmbjGGfb0De-HKD0t2PSH-NTRErK13TcugXAGL7LXxrTxpNQy7335UaOEfGsKTL54FZDNydvISdqvgqBebqg/s435/1,3.JPG)

### Adicional

Al estar configurando una VSI en equipos Huawei podemos agregar algunos parámetros que son muy útiles en varios casos:

* *mac-withdraw enable* sirve para que el equipo que lo tenga configurado cambie o elimine una dirección MAC
* *upe-npe mac-withdraw enable* reenvía mensaje a otros nodos sobre algun cambio o eliminación de una dirección física
* *npe-upe mac-withdraw enable* similar al anterior reenvía mensajes de MAC retiradas pero espera a que reciba otro mismo mensaje para hacerlo efectivo
* *upe-upe mac-withdraw enable* reenvía mensaje de MAC retirada cuando recibe de varios nodos 
* *peer _ip_ negotiation-vc-id _vsi-id_* hace posible la comunicación efectiva de VPLS distintas al poner un vsi-id local y otro vsi-id para que pueda negociarlo y conocerlo

Estas van dentro de la VSI:
```html
 pwsignal ldp
  vsi-id 20
  mac-withdraw enable
  upe-npe mac-withdraw enable
  peer 10.90.214.1 negotiation-vc-id 10
#
return
```
Si tenemos algún switch L3 y queremos hacer una VPLS incluyendo una VLAN debemos habilitar el tipo de vlan-vpn en ella
```html
Habilitar vlan-vpn en un puerto
    vlan-vpn enable

Deshabilitar vlan-vpn en un puerto
    undo vlan-vpn
```
## [Atrás](./)