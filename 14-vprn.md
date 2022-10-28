---
title: VPRN
layout: default
---

*Virtual Private Routed Network* es un tipo de red multipunto a multipunto que actúa en capa 3 que busca formar una arquitectura de routing lógica e idependiente, esta se basa también sobre una red IP/MPLS. Una *VPRN* consiste de un grupo clientes conectados a los equipos de un proveedor y cada uno de estos equipos del proveedor tiene una tabla de reenvío de IP separada para cada VPRN configurada en los equipos.

### Configuración

Los nodos usados para la VPRN son equipos Nokia hasta la capa de acceso, en equipos de los clientes hay de diferentes proveedores. R1 Fortigate, R2 Huawei AR100, R3 Cisco. Estaremos trabajando en base a esta topología

![a1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEha2-AF1z7kyb_HFfOFQ-hOOEKdp53GtWeJUfIuVHvOhov1XT9Te2Ot-I89tKTrfy1lhLad8gB_5X2g4-9UKGnfL14weLPltnbaWKwXXtO1ixIjMRYxHSVAfS5tiT-TdSX16YFCdjphAHjZ0qKlgB7Fmxjp4t56sLqTICqgVFoWnvWImDlgmD2p4nL2CA/s1334/Otros%20(1).jpeg)

En cada uno de los equipos Nokia se configura como protocolo de enrutamiento IS-IS level 1, en caso de tener otras áreas siempre sobre este protocolo se debe configurar los nodos con el nivel que corresponda, por ejemplo si solo se configura IS-IS en varios equipos y todos estos son de una única área son de *level 1* para IS-IS, si tenemos más áreas en nuestra red los equipos borde se configuran con *level 2* que es un tipo de red troncal que puede comunicarse con vecinos de otras áreas, estos equipos conocen *LSBD* de las áreas con las que junten, y si tenemos de forma similar varias áreas pero los routers borde también es necesario que estén en un aréa se configura para ambos niveles *level 1-2* que va a ser posible que este router tenga vecinos de cualquier área y también va a tener diferentes *LSBD* para cada nivel (1 y 2).

```html
A:Core-1>config>router>isis$ info
----------------------------------------------
        level-capability level-1
        area-id 10.10
        authentication-key "Yh3EFPp/XdVvThImCq9w3n4zev9vTT2s" hash2
        authentication-type password
        reference-bandwidth 100000000
        level 1
            wide-metrics-only
        exit
        interface "system"
            no shutdown
        exit
        interface "to-Aggre-1"
            interface-type point-to-point
            no shutdown
        exit
        interface "to-Core-2"
            interface-type point-to-point
            no shutdown
        exit
        no shutdown
----------------------------------------------

A:Core-2>config>router>isis$ info
----------------------------------------------
        level-capability level-1
        area-id 10.10
        authentication-key "Yh3EFPp/XdVvThImCq9w3n4zev9vTT2s" hash2
        authentication-type password
        reference-bandwidth 100000000
        level 1
            wide-metrics-only
        exit
        interface "system"
            no shutdown
        exit
        interface "to-Aggre-2"
            interface-type point-to-point
            no shutdown
        exit
        interface "to-Core-1"
            interface-type point-to-point
            no shutdown
        exit
        no shutdown
```

Una vez configurado en todos los nodos, procedemos a verificar adyacencias y comunicación
```html
A:Aggre-1>config>router>isis$ show router isis database

===============================================================================
Router Base ISIS Instance 0 Database
===============================================================================
LSP ID                                  Sequence  Checksum Lifetime Attributes
-------------------------------------------------------------------------------

Displaying Level 1 database
-------------------------------------------------------------------------------
Core-1.00-00                            0xc       0xa3c6   661      L1
Core-2.00-00                            0xa       0xbe75   805      L1
Aggre-1.00-00                           0xb       0xb69e   663      L1
Aggre-2.00-00                           0x8       0xbe3e   805      L1

A:Aggre-2>config>router>isis$ show router isis topology ipv4-unicast detail

===============================================================================
Router Base ISIS Instance 0 Topology Table
===============================================================================
-------------------------------------------------------------------------------
IS-IS IP paths (MT-ID 0), Level 1
-------------------------------------------------------------------------------
Node      : Core-1.00                        Metric      : 200
Interface : to-Core-2                        SNPA        : none
Nexthop   : Core-2

Node      : Core-2.00                        Metric      : 100
Interface : to-Core-2                        SNPA        : none
Nexthop   : Core-2

Node      : Aggre-1.00                       Metric      : 300
Interface : to-Core-2                        SNPA        : none
Nexthop   : Core-2

===============================================================================

A:Acc-1# show router isis adjacency detail

==============================================================================
Router Base ISIS Instance 0 Adjacency
==============================================================================
SystemID    : Aggre-1                          SNPA        : 50:00:00:01:00:02
Interface   : to-Aggr-1                        Up Time     : 0d 00:06:38
State       : Up                               Priority    : 0
Nbr Sys Typ : L1                               L. Circ Typ : L1
Hold Time   : 25                               Max Hold    : 27
Adj Level   : L1                               MT Enabled  : No
Topology    : Unicast

IPv6 Neighbor     : ::
IPv4 Neighbor     : 10.90.214.1
Restart Support   : Disabled
Restart Status    : Not currently being helped
Restart Supressed : Disabled
Number of Restarts: 0
Last Restart at   : Never

==============================================================================

A:Acc-1# ping 3.3.3.3 count 1
PING 3.3.3.3 56 data bytes
64 bytes from 3.3.3.3: icmp_seq=1 ttl=64 time=6.22ms.

---- 3.3.3.3 PING Statistics ----
1 packet transmitted, 1 packet received, 0.00% packet loss
round-trip min = 6.22ms, avg = 6.22ms, max = 6.22ms, stddev = 0.000ms

A:Acc-2# show router isis adjacency

===============================================================================
Router Base ISIS Instance 0 Adjacency
===============================================================================
System ID                Usage State Hold Interface                     MT-ID
-------------------------------------------------------------------------------
Aggre-2                  L1    Up    21   to-Aggre-2                    0
-------------------------------------------------------------------------------
Adjacencies : 1
===============================================================================

A:Acc-2# ping count 1 4.4.4.4
PING 4.4.4.4 56 data bytes
64 bytes from 4.4.4.4: icmp_seq=1 ttl=64 time=2.41ms.

---- 4.4.4.4 PING Statistics ----
1 packet transmitted, 1 packet received, 0.00% packet loss
round-trip min = 2.41ms, avg = 2.41ms, max = 2.41ms, stddev = 0.000ms
```
Ahora toca configurar la red para SDP en los puntos finales tener *MPLS* en la red, cabe mencionar que para que podamos ver adyacencias por *LDP* debemos comprobar que tengamos routing en nuestros nodos y comprobar comunicación con los puntos finales, en este caso SDP estará apuntando a las direcciones IP de las interfaces del sistema de cada nodo dentro de este proceso.

Configuramos *SDP* ya que es el que nos permitirá formar sesiones en los puntos finales y así estos puedan tener una comunicación correcta y efectiva.

![a2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjGmGo2Wl-W2zOGbdIrlcvNnEBvV-1pG8_ReGC1D9rR4oyE8nVNp16-hwO28qeNRcKVuoh7Vl954ReJuBk8qOcSszNlBm1OQTya__1NKde4PlVG58y2TKnmrzrODH2tp0arvQE5hISaVF3ufppdSsk0WhnNfFNR_BObIODGWoIF80X1AMf2OfFwY0_1oA/s1496/Otros.jpeg)

> Es importante configurar mpls y ldp en cada interfaz de los nodos para que nuestra red *IP/MPLS* pueda trabajar sin problemas, esta va habilitado en cada interfaz que conecta al otro cliente es decir cada nodo que está involucrado en la red VPRN y esto lo habilitamos usando lo siguiente:

```bash
    configure router mpls interface to-Aggre-1 no shutdown
    configure router mpls interface system no shutdown
    configure router mpls no shutdown
    configure router ldp interface-parameters interface to-Aggre-1 no shutdown
    configure router ldp no shutdown
```

La ip que lleva cada nodo en las interfaces "system" van de forma secuencial, el router Core-1 tiene en su interfaz la dirección IP 1.1.1.1, Core-2 2.2.2.2, Aggre.-1 3.3.3.3 y continuan así hasta llegar al equipo Aggre.-3 con la dirección 7.7.7.7

```html
*A:Acc-1>config>service# info
----------------------------------------------
        sdp 17006 mpls create
            far-end 6.6.6.6
            ldp
            keep-alive
                shutdown
            exit
            no shutdown
        exit
        sdp 17007 mpls create
            far-end 7.7.7.7
            ldp
            keep-alive
                shutdown
            exit
            no shutdown
        exit
```
```html
A:Acc-2>config>service# info
----------------------------------------------
        sdp 17005 mpls create
            far-end 5.5.5.5
            ldp
            keep-alive
                shutdown
            exit
            no shutdown
        exit
        sdp 17007 mpls create
            far-end 7.7.7.7
            ldp
            keep-alive
                shutdown
            exit
            no shutdown
        exit
```
```html

A:Aggre-3>config>service# info
----------------------------------------------
        sdp 17005 mpls create
            far-end 5.5.5.5
            ldp
            keep-alive
                shutdown
            exit
            no shutdown
        exit
        sdp 17006 mpls create
            far-end 6.6.6.6
            ldp
            keep-alive
                shutdown
            exit
            no shutdown
        exit
```

Ahora para verificar que nuestros *SDP* estén arriba (up) podemos probar comunicarnos de punta a punta para que forma la unión

```bash
*A:Acc-2# ping 5.5.5.5 count 1
PING 5.5.5.5 56 data bytes
64 bytes from 5.5.5.5: icmp_seq=1 ttl=60 time=8.18ms.

---- 5.5.5.5 PING Statistics ----
1 packet transmitted, 1 packet received, 0.00% packet loss
round-trip min = 8.18ms, avg = 8.18ms, max = 8.18ms, stddev = 0.000ms

*A:Acc-2# show service sdp

============================================================================
Services: Service Destination Points
============================================================================
SdpId  AdmMTU  OprMTU  Far End          Adm  Opr         Del     LSP   Sig
----------------------------------------------------------------------------
17005  0       8914    5.5.5.5          Up   Up          MPLS    L     TLDP
17007  0       0       7.7.7.7          Up   Down        MPLS    L     TLDP
----------------------------------------------------------------------------
Number of SDPs : 2
----------------------------------------------------------------------------
Legend: R = RSVP, L = LDP, B = BGP, M = MPLS-TP, n/a = Not Applicable
============================================================================
```

Vemos que ya esta Operativo nuestro SDP solo queda hacer los mismo con los otros nodos para que veamos todo 'Up'

### VPRN

Iniciamos configurando a el cliente en cada nodo, y luego asociamos este cliente a la VPRN, es importante esto ya que de lo contrario la VPRN no tendrá un cliente asociado a ella

```html
A:Acc-1>config>service#info
        customer 55 create
            description "Client-LA-US"
            contact "admin@clientla.com"
            phone "12064563059"
        vprn 5511 customer 40 create
            router-id 192.168.1.0
            autonomous-system 45000
            route-distinguisher 45000:5511
            vrf-target target:45000:5511
            interface "to-R1" create
                address 192.168.1.0/31
                sap 1/1/3:55 create
                exit
            exit
            no shutdown
```
```html
A:Acc-2>config>service#info
        vprn 5511 customer 55 create
            router-id 192.168.2.0
            autonomous-system 45000
            route-distinguisher 45000:5511
            vrf-target target:45000:5511
            interface "to-R2" create
                description "Customer_NY_USA"
                address 192.168.2.0/31
                sap 1/1/3:55 create
                exit
            exit
            spoke-sdp 17005 create
                description "to-Acc-1"
            exit
            spoke-sdp 17007 create
                description "to-Aggre-3"
            exit
            no shutdown
        exit
```
```html
A:Aggre-3>config>service# info
----------------------------------------------
        customer 1 create
            description "Default customer"
        exit
        customer 55 create
            description "Customer_LA_USA.Inc"
            contact "admin@la-us.com"
            phone "1-2027953213"
        exit
        vprn 5511 customer 55 create
            router-id 192.168.3.0
            autonomous-system 45000
            route-distinguisher 45000:5511
            vrf-target target:45000:5511
            interface "to-R3" create
                description "Customer_LA-USA"
                address 192.168.3.0/31
                sap 1/1/3:55 create
                exit
            exit
            no shutdown
        exit
```
Ya configurado VPRN en los nodos, podemos empezar a configurar los equipos de los clientes en cada 'Site'

##### R1

Configuraciones en el Firewall Fortigate:
```html
FortiGate-VM64-KVM # config system global
FortiGate-VM64-KVM (global) # set hostname 55-CHA-USA
FortiGate-VM64-KVM (global) # end
55-CHA-USA # config system interface
55-CHA-USA (interface) # edit vlan_55
55-CHA-USA (vlan_55) # set type vlan
55-CHA-USA (vlan_55) # set vlanid 100
55-CHA-USA (vlan_55) # set interface port1
55-CHA-USA (vlan_55) # set mode static
55-CHA-USA (vlan_55) # set ip 192.168.1.1 255.255.255.254
55-CHA-USA (vlan_55) # set allowaccess ssh https http ping ftm
55-CHA-USA (vlan_55) # set vdom root
55-CHA-USA (vlan_55) # end

55-CHA-USA # get system interface
== [ port1 ]
name: port1   mode: static    ip: 0.0.0.0 0.0.0.0   status: up    netbios-forward: disable    type: physical   ring-rx: 0   ring-tx: 0   netflow-sampler: disable    sflow-sampler: disable    src-check: enable    explicit-web-proxy: disable    explicit-ftp-proxy: disable    proxy-captive-portal: disable    mtu-override: disable    wccp: disable    drop-overlapped-fragment: disable    drop-fragment: disable
== [ port2 ]
name: port2   mode: dhcp    ip: 192.168.40.159 255.255.255.0   status: up    netbios-forward: disable    type: physical   ring-rx: 0   ring-tx: 0   netflow-sampler: disable    sflow-sampler: disable    src-check: enable    explicit-web-proxy: disable    explicit-ftp-proxy: disable    proxy-captive-portal: disable    mtu-override: disable    wccp: disable    drop-overlapped-fragment: disable    drop-fragment: disable
== [ port3 ]
name: port3   mode: static    ip: 192.168.5.1 255.255.255.0   status: up    netbios-forward: disable    type: physical   ring-rx: 0   ring-tx: 0   netflow-sampler: disable    sflow-sampler: disable    src-check: enable    explicit-web-proxy: disable    explicit-ftp-proxy: disable    proxy-captive-portal: disable    mtu-override: disable    wccp: disable    drop-overlapped-fragment: disable    drop-fragment: disable
== [ vlan_55 ]
name: vlan_55   mode: static    ip: 192.168.1.1 255.255.255.254   status: up    netbios-forward: disable    type: vlan   netflow-sampler: disable    sflow-sampler: disable    src-check: enable    explicit-web-proxy: disable    explicit-ftp-proxy: disable    proxy-captive-portal: disable    switch-controller-feature: none    mtu-override: disable    wccp: disable    drop-overlapped-fragment: disable    drop-fragment: disable

55-CHA-USA # exec ping 192.168.1.0
PING 192.168.1.0 (192.168.1.0): 56 data bytes
64 bytes from 192.168.1.0: icmp_seq=0 ttl=64 time=6.8 ms
64 bytes from 192.168.1.0: icmp_seq=1 ttl=64 time=2.3 ms
64 bytes from 192.168.1.0: icmp_seq=2 ttl=64 time=1.5 ms
64 bytes from 192.168.1.0: icmp_seq=3 ttl=64 time=1.9 ms
64 bytes from 192.168.1.0: icmp_seq=4 ttl=64 time=2.1 ms

--- 192.168.1.0 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.5/2.9/6.8 ms

55-CHA-USA # exec traceroute 192.168.1.0
traceroute to 192.168.1.0 (192.168.1.0), 32 hops max, 3 probe packets per hop, 84 byte packets
 1  192.168.1.0  1.654 ms  1.750 ms  1.254 ms

55-CHA-USA # get system arp
Address           Age(min)   Hardware Addr      Interface
192.168.1.0       0          50:00:00:05:00:03 vlan_55
```

Configuramos una vlan en el Firewall, que es la correspondiente al cliente, y como en el equipo de acceso se configuró el puerto físico para que trabaje en modo de acceso y encapsulamiento y así poder tener a más clientes conectados

```html
configure port 1/1/3
        description "to-R1"
        ethernet
            mode access
            encap-type dot1q
        exit
        no shutdown
```

Para el lado del cliente LAN del Fortigate se configura dhcp para que los clientes se puedan conectar de forma práctica

```html
55-CHA-USA # config system dhcp server
55-CHA-USA (server) # edit 1
55-CHA-USA (1) # set auto-configuration disable
55-CHA-USA (1) # set dns-service default
55-CHA-USA (1) # set interface port3
55-CHA-USA (1) # set default-gateway 192.168.5.1
55-CHA-USA (1) # config ip-range
55-CHA-USA (ip-range) # edit 1
55-CHA-USA (1) # set start-ip 192.168.5.10
55-CHA-USA (1) # set end-ip 192.168.5.254
55-CHA-USA (1) # next
55-CHA-USA (ip-range) # end
55-CHA-USA (1) # set netmask 255.255.255.0
55-CHA-USA (1) # next
55-CHA-USA (server) # end
```

Todas estas configuraciones que se hicieron por línea de comandos lo podemos hacer de forma gráfica ya que Fortinet nos da esa opción en sus equipos, para ello si usamos algún emulador podemos agregar una nueva red o nube y hacer que en la interfaz que conecta a esta nube se pueda acceder por medio de un protocolo web, en caso de no hacerlo en emulador es conectarse al puerto de consola del equipo y por algún gestor de conexiones como Putty, Xterm, SecureCRT iniciar a configurarlo:

```html
FortiGate-VM64-KVM # config system interface
FortiGate-VM64-KVM (interface) # edit port2
FortiGate-VM64-KVM (port2) # set allowaccess http https ssh ping
FortiGate-VM64-KVM (port2) # set mode dhcp
FortiGate-VM64-KVM (port2) # set status up
FortiGate-VM64-KVM (port2) # end
FortiGate-VM64-KVM # get system interface
== [ port2 ]
name: port2   mode: dhcp    ip: 192.168.40.159 255.255.255.0   status: up    netbios-forward: disable    type: physical   ring-rx: 0   ring-tx: 0   netflow-sampler: disable    sflow-sampler: disable    src-check: enable    explicit-web-proxy: disable    explicit-ftp-proxy: disable    proxy-captive-portal: disable    mtu-override: disable    wccp: disable    drop-overlapped-fragment: disable    drop-fragment: disable
```

Con la ip que asigna podemos o bien nosotros le asignemos la ponemos en algún navegador y veremos la interfaz del equipo

##### R2

Configuraciones en Huawei AR100:
```html
[R2-vpn-instance-Cust_2-af-ipv4]disp this
#
 ipv4-family
  route-distinguisher 45002:5511
  vpn-target 45002:5511 export-extcommunity
  vpn-target 45002:5511 import-extcommunity
#
<R2>disp current-configuration interface giga 0/0/0.55
#
interface GigabitEthernet0/0/0.55
 dot1q termination vid 55
 ip binding vpn-instance Cust_2
 ip address 192.168.2.1 255.255.255.254
#

<R2>disp arp all
IP ADDRESS      MAC ADDRESS     EXPIRE(M) TYPE        INTERFACE   VPN-INSTANCE
                                    VLAN/CEVLAN(SIP/DIP)      PVC
------------------------------------------------------------------------------
192.168.2.1     5000-000a-0000            I -         GE0/0/0.55     Cust_2     
192.168.2.0     5000-0006-0003  18        D-0         GE0/0/0.55     Cust_2     
                                            55/-
------------------------------------------------------------------------------
Total:2         Dynamic:1       Static:0     Interface:1

<R2>ping -vpn-instance Cust_2 192.168.2.0
  PING 192.168.2.0: 56  data bytes, press CTRL_C to break
    Reply from 192.168.2.0: bytes=56 Sequence=1 ttl=64 time=2 ms
    Reply from 192.168.2.0: bytes=56 Sequence=2 ttl=64 time=2 ms
    Reply from 192.168.2.0: bytes=56 Sequence=3 ttl=64 time=8 ms
    Reply from 192.168.2.0: bytes=56 Sequence=4 ttl=64 time=2 ms
    Reply from 192.168.2.0: bytes=56 Sequence=5 ttl=64 time=7 ms

  --- 192.168.2.0 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 2/4/8 ms
```
En este equipo configuramos la instancia VPN llamada Cust_2, el router distinguisher es 45002 porque debe ser diferente AS para cada sitio, en caso que queramos que 2 puntos diferentes tengan comunicación se debe configurar el mismo AS para cada equipo, en este caso son 3 puntos distintos con AS independiente, el objetivo (target) debe ser el AS:VPRN configurada en los nodos

##### R3

Configuraciones en equipo Cisco:
```html
R3#show running-config | section vrf
vrf definition 5511
 rd 45003:5511
 !
 address-family ipv4
  route-target export 45003:5511
  route-target import 45003:5511
 exit-address-family

R3#show running-config interface ethernet 0/0.55
!
interface Ethernet0/0.55
 encapsulation dot1Q 55
 vrf forwarding 5511
 ip address 192.168.3.1 255.255.255.254
end
```
Las configuraciones son similares que R2, comprobamos comunicación:
```html
R3#ping vrf 5511 192.168.3.0 repeat 15
Type escape sequence to abort.
Sending 15, 100-byte ICMP Echos to 192.168.3.0, timeout is 2 seconds:
!!!!!!!!!!!!!!!
Success rate is 100 percent (15/15), round-trip min/avg/max = 1/1/2 ms
R3#show arp vrf 5511
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.3.0             0   5000.000c.0003  ARPA   Ethernet0/0.55
Internet  192.168.3.1             -   aabb.cc00.b000  ARPA   Ethernet0/0.55
```

#### Comprobaciones

Podemos probar con diversos comandos el estado de nuestras configuraciones, primero podemos usar los comandos de *oam* el primero a utilizar es *svc-ping* que comprueba que el servicio configurado en los puntos finales tengan un aprovisionamiento correcto, por defecto este comando solo envía una petición

```html

A:Acc-1# oam svc-ping 6.6.6.6 service 5511 local-sdp remote-sdp
Service-ID: 5511

Err Info                Local           Remote
-----------------------------------------------------
    Type:               VPRN            VPRN
    Admin State:        Up              Up
    Oper State:         Up              Up
    Service-MTU:        0               0
    Customer ID:        40              55

    IP Interface State: Up
    Actual IP Addr:     5.5.5.5         6.6.6.6
    Expected Peer IP:   6.6.6.6         5.5.5.5

    SDP Path Used:      No              No
    SDP-ID:             17006           17005
    Admin State:        Up              Up
    Operative State:    Up              Up
    Binding Admin State:Up              Up
    Binding Oper State: Up              Up
    Binding VC ID:      5511            5511
    Binding Type:       Spoke           Spoke
    Binding Vc-type:    N/A             N/A
    Binding Vlan-vc-tag:N/A             N/A

    Egress Label:       0               0
    Ingress Label:      0               0
    Egress Label Type:  Signaled        Signaled
    Ingress Label Type: Signaled        Signaled

Request Result: Sent - Reply Received

A:Acc-2# oam svc-ping 7.7.7.7 service 5511 local-sdp remote-sdp
Service-ID: 5511

Err Info                Local           Remote
-----------------------------------------------------
    Type:               VPRN            VPRN
    Admin State:        Up              Up
    Oper State:         Up              Up
    Service-MTU:        0               0
    Customer ID:        55              55

    IP Interface State: Up
    Actual IP Addr:     6.6.6.6         7.7.7.7
    Expected Peer IP:   7.7.7.7         6.6.6.6

    SDP Path Used:      No              No
    SDP-ID:             17007           17006
    Admin State:        Up              Up
    Operative State:    Up              Up
    Binding Admin State:Up              Up
    Binding Oper State: Up              Up
    Binding VC ID:      5511            5511
    Binding Type:       Spoke           Spoke
    Binding Vc-type:    N/A             N/A
    Binding Vlan-vc-tag:N/A             N/A

    Egress Label:       0               0
    Ingress Label:      0               0
    Egress Label Type:  Signaled        Signaled
    Ingress Label Type: Signaled        Signaled

Request Result: Sent - Reply Received

A:Aggre-3# oam svc-ping 6.6.6.6 service 5511 local-sdp remote-sdp
Service-ID: 5511

Err Info                Local           Remote
-----------------------------------------------------
    Type:               VPRN            VPRN
    Admin State:        Up              Up
    Oper State:         Up              Up
    Service-MTU:        0               0
    Customer ID:        55              55

    IP Interface State: Up
    Actual IP Addr:     7.7.7.7         6.6.6.6
    Expected Peer IP:   6.6.6.6         7.7.7.7

    SDP Path Used:      No              No
    SDP-ID:             17006           17007
    Admin State:        Up              Up
    Operative State:    Up              Up
    Binding Admin State:Up              Up
    Binding Oper State: Up              Up
    Binding VC ID:      5511            5511
    Binding Type:       Spoke           Spoke
    Binding Vc-type:    N/A             N/A
    Binding Vlan-vc-tag:N/A             N/A

    Egress Label:       0               0
    Ingress Label:      0               0
    Egress Label Type:  Signaled        Signaled
    Ingress Label Type: Signaled        Signaled

Request Result: Sent - Reply Received
```

Ahora podemos verificar que número de etiqueta están manejando los equipos, para ello ejecutamos:

```html
A:Acc-1# show router ldp bindings

===============================================================================
LDP LSR ID: 5.5.5.5
===============================================================================
Legend: U - Label In Use, N - Label Not In Use, W - Label Withdrawn
        S - Status Signaled Up,  D - Status Signaled Down
        E - Epipe Service, V - VPLS Service, M - Mirror Service
        A - Apipe Service, F - Fpipe Service, I - IES Service, R - VPRN service
        P - Ipipe Service, WP - Label Withdraw Pending, C - Cpipe Service
        BU - Alternate For Fast Re-Route, TLV - (Type, Length: Value)
===============================================================================
LDP Prefix Bindings
===============================================================================
Prefix              IngLbl       EgrLbl     EgrIntf/         EgrNextHop
   Peer                                     LspId
-------------------------------------------------------------------------------
1.1.1.1/32          131068N      131069     1/1/2            10.90.214.1
   3.3.3.3
2.2.2.2/32          131067N      131068     1/1/2            10.90.214.1
   3.3.3.3
3.3.3.3/32            --         131071     1/1/2            10.90.214.1
   3.3.3.3
4.4.4.4/32          131066N      131067     1/1/2            10.90.214.1
   3.3.3.3
5.5.5.5/32          131071U        --         --               --
   3.3.3.3
6.6.6.6/32          131065N      131066     1/1/2            10.90.214.1
   3.3.3.3
7.7.7.7/32          131064N      131065     1/1/2            10.90.214.1
   3.3.3.3
-------------------------------------------------------------------------------
No. of Prefix Bindings: 7
===============================================================================

A:Acc-1# show router ldp bindings active

===============================================================================
Legend:  (S) - Static       (M) - Multi-homed Secondary Support
         (B) - BGP Next Hop (BU) - Alternate Next-hop for Fast Re-Route
===============================================================================
LDP Prefix Bindings (Active)
===============================================================================
Prefix                  Op   IngLbl    EgrLbl    EgrIntf/LspId  EgrNextHop
-------------------------------------------------------------------------------
1.1.1.1/32              Push   --      131069    1/1/2          10.90.214.1
1.1.1.1/32              Swap 131068    131069    1/1/2          10.90.214.1
2.2.2.2/32              Push   --      131068    1/1/2          10.90.214.1
2.2.2.2/32              Swap 131067    131068    1/1/2          10.90.214.1
3.3.3.3/32              Push   --      131071    1/1/2          10.90.214.1
4.4.4.4/32              Push   --      131067    1/1/2          10.90.214.1
4.4.4.4/32              Swap 131066    131067    1/1/2          10.90.214.1
5.5.5.5/32              Pop  131071      --        --             --
6.6.6.6/32              Push   --      131066    1/1/2          10.90.214.1
6.6.6.6/32              Swap 131065    131066    1/1/2          10.90.214.1
7.7.7.7/32              Push   --      131065    1/1/2          10.90.214.1
7.7.7.7/32              Swap 131064    131065    1/1/2          10.90.214.1
-------------------------------------------------------------------------------
No. of Prefix Active Bindings: 12
===============================================================================

===============================================================================
LDP LSR ID: 5.5.5.5
===============================================================================
Legend: U - Label In Use,  N - Label Not In Use, W - Label Withdrawn
        WP - Label Withdraw Pending, BU - Alternate For Fast Re-Route
===============================================================================
LDP Generic P2MP Bindings (Active)
===============================================================================
```

Con este output vemos que solo hay una etiqueta en uso que es *131071* y si miramos en el otro equipo, tenemos estos resultados:

```html
A:Acc-2# show router ldp bindings detail

===============================================================================
LDP LSR ID: 6.6.6.6
===============================================================================
Legend: U - Label In Use, N - Label Not In Use, W - Label Withdrawn
        S - Status Signaled Up,  D - Status Signaled Down
        E - Epipe Service, V - VPLS Service, M - Mirror Service
        A - Apipe Service, F - Fpipe Service, I - IES Service, R - VPRN service
        P - Ipipe Service, WP - Label Withdraw Pending, C - Cpipe Service
        BU - Alternate For Fast Re-Route, TLV - (Type, Length: Value)
===============================================================================
LDP Prefix Bindings
===============================================================================
-------------------------------------------------------------------------------
Prefix             : 1.1.1.1/32
-------------------------------------------------------------------------------
Ing Lbl            : 131067N             Peer                : 4.4.4.4
Egr Lbl            : 131068
Egr Int/LspId      : 1/1/2
EgrNextHop         : 10.90.215.1
Egr. Flags         : None                Ing. Flags          : None
Egr If Name        : to-Aggre-2
Metric             : 300                 Mtu                 : 8922
-------------------------------------------------------------------------------
Prefix             : 2.2.2.2/32
-------------------------------------------------------------------------------
Ing Lbl            : 131068N             Peer                : 4.4.4.4
Egr Lbl            : 131069
Egr Int/LspId      : 1/1/2
EgrNextHop         : 10.90.215.1
Egr. Flags         : None                Ing. Flags          : None
Egr If Name        : to-Aggre-2
Metric             : 200                 Mtu                 : 8922
-------------------------------------------------------------------------------
Prefix             : 3.3.3.3/32
-------------------------------------------------------------------------------
Ing Lbl            : 131066N             Peer                : 4.4.4.4
Egr Lbl            : 131067
Egr Int/LspId      : 1/1/2
EgrNextHop         : 10.90.215.1
Egr. Flags         : None                Ing. Flags          : None
Egr If Name        : to-Aggre-2
Metric             : 400                 Mtu                 : 8922
-------------------------------------------------------------------------------
Prefix             : 4.4.4.4/32
-------------------------------------------------------------------------------
Ing Lbl            :   --                Peer                : 4.4.4.4
Egr Lbl            : 131071
Egr Int/LspId      : 1/1/2
EgrNextHop         : 10.90.215.1
Egr. Flags         : None                Ing. Flags          : None
Egr If Name        : to-Aggre-2
Metric             : 100                 Mtu                 : 8922
-------------------------------------------------------------------------------
Prefix             : 5.5.5.5/32
-------------------------------------------------------------------------------
Ing Lbl            : 131065N             Peer                : 4.4.4.4
Egr Lbl            : 131066
Egr Int/LspId      : 1/1/2
EgrNextHop         : 10.90.215.1
Egr. Flags         : None                Ing. Flags          : None
Egr If Name        : to-Aggre-2
Metric             : 500                 Mtu                 : 8922
-------------------------------------------------------------------------------
Prefix             : 6.6.6.6/32
-------------------------------------------------------------------------------
Ing Lbl            : 131071U             Peer                : 4.4.4.4
Egr Lbl            :   --
Egr Int/LspId      :   --
EgrNextHop         :   --
Egr. Flags         : None                Ing. Flags          : None
-------------------------------------------------------------------------------
Prefix             : 7.7.7.7/32
-------------------------------------------------------------------------------
Ing Lbl            : 131064N             Peer                : 4.4.4.4
Egr Lbl            : 131065
Egr Int/LspId      : 1/1/2
EgrNextHop         : 10.90.215.1
Egr. Flags         : None                Ing. Flags          : None
Egr If Name        : to-Aggre-2
Metric             : 400                 Mtu                 : 8922
===============================================================================
No. of Prefix Bindings: 7
===============================================================================

A:Acc-2# show router ldp bindings active

===============================================================================
Legend:  (S) - Static       (M) - Multi-homed Secondary Support
         (B) - BGP Next Hop (BU) - Alternate Next-hop for Fast Re-Route
===============================================================================
LDP Prefix Bindings (Active)
===============================================================================
Prefix                  Op   IngLbl    EgrLbl    EgrIntf/LspId  EgrNextHop
-------------------------------------------------------------------------------
1.1.1.1/32              Push   --      131068    1/1/2          10.90.215.1
1.1.1.1/32              Swap 131067    131068    1/1/2          10.90.215.1
2.2.2.2/32              Push   --      131069    1/1/2          10.90.215.1
2.2.2.2/32              Swap 131068    131069    1/1/2          10.90.215.1
3.3.3.3/32              Push   --      131067    1/1/2          10.90.215.1
3.3.3.3/32              Swap 131066    131067    1/1/2          10.90.215.1
4.4.4.4/32              Push   --      131071    1/1/2          10.90.215.1
5.5.5.5/32              Push   --      131066    1/1/2          10.90.215.1
5.5.5.5/32              Swap 131065    131066    1/1/2          10.90.215.1
6.6.6.6/32              Pop  131071      --        --             --
7.7.7.7/32              Push   --      131065    1/1/2          10.90.215.1
7.7.7.7/32              Swap 131064    131065    1/1/2          10.90.215.1
-------------------------------------------------------------------------------
No. of Prefix Active Bindings: 12
===============================================================================

===============================================================================
LDP LSR ID: 6.6.6.6
===============================================================================
Legend: U - Label In Use,  N - Label Not In Use, W - Label Withdrawn
        WP - Label Withdraw Pending, BU - Alternate For Fast Re-Route
===============================================================================
LDP Generic P2MP Bindings (Active)
===============================================================================
```

Y de igual forma Access-2 usa la misma etiqueta para comunicarse, y viendo el estado de las asociaciones con *show router ldp bindings active* está etiqueta *131071* actúa como Pop que lo que hace es eliminar la carga superior reenvíar la carga útil restante. El equipo Aggregate-3 de igual forma usa la misma etiqueta

```html
A:Aggre-3# show router ldp bindings

===============================================================================
LDP LSR ID: 7.7.7.7
===============================================================================
Legend: U - Label In Use, N - Label Not In Use, W - Label Withdrawn
        S - Status Signaled Up,  D - Status Signaled Down
        E - Epipe Service, V - VPLS Service, M - Mirror Service
        A - Apipe Service, F - Fpipe Service, I - IES Service, R - VPRN service
        P - Ipipe Service, WP - Label Withdraw Pending, C - Cpipe Service
        BU - Alternate For Fast Re-Route, TLV - (Type, Length: Value)
===============================================================================
LDP Prefix Bindings
===============================================================================
Prefix              IngLbl       EgrLbl     EgrIntf/         EgrNextHop
   Peer                                     LspId
-------------------------------------------------------------------------------
1.1.1.1/32            --         131071     1/1/2            10.204.104.1
   1.1.1.1
2.2.2.2/32          131067N      131069     1/1/2            10.204.104.1
   1.1.1.1
3.3.3.3/32          131068N      131070     1/1/2            10.204.104.1
   1.1.1.1
4.4.4.4/32          131066N      131068     1/1/2            10.204.104.1
   1.1.1.1
6.6.6.6/32          131065N      131067     1/1/2            10.204.104.1
   1.1.1.1
7.7.7.7/32          131071U        --         --               --
   1.1.1.1
-------------------------------------------------------------------------------
No. of Prefix Bindings: 6
===============================================================================

*A:Aggre-3# show router ldp bindings active

===============================================================================
Legend:  (S) - Static       (M) - Multi-homed Secondary Support
         (B) - BGP Next Hop (BU) - Alternate Next-hop for Fast Re-Route
===============================================================================
LDP Prefix Bindings (Active)
===============================================================================
Prefix                  Op   IngLbl    EgrLbl    EgrIntf/LspId  EgrNextHop
-------------------------------------------------------------------------------
1.1.1.1/32              Push   --      131071    1/1/2          10.204.104.1
2.2.2.2/32              Push   --      131069    1/1/2          10.204.104.1
2.2.2.2/32              Swap 131067    131069    1/1/2          10.204.104.1
3.3.3.3/32              Push   --      131070    1/1/2          10.204.104.1
3.3.3.3/32              Swap 131068    131070    1/1/2          10.204.104.1
4.4.4.4/32              Push   --      131068    1/1/2          10.204.104.1
4.4.4.4/32              Swap 131066    131068    1/1/2          10.204.104.1
6.6.6.6/32              Push   --      131067    1/1/2          10.204.104.1
6.6.6.6/32              Swap 131065    131067    1/1/2          10.204.104.1
7.7.7.7/32              Pop  131071      --        --             --
-------------------------------------------------------------------------------
No. of Prefix Active Bindings: 10
===============================================================================
```

Ahora podemos también ver que etiqueta usa y la forma en que va el paquete capturando el tráfico con Wireshark o Tshark desde consola

![a3](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiOq-6QIZ1j47fVaqGUZVoJTr4ugcv_iKNFe_BTa1rVnVG8k1fC_GJOk40_2yMgoricuJjopLxinqPmsvp_jSPdBYTEM8tJS9BWs8qc8atq-Mu3nOHQJRZF1oqpxUyKm_oJXfxHom-M06OnxTvc1EbmCZdQPDm4LX7c5VHThv3vexSeg97oqCaZLH_YKg/s808/Captura.jpg)

## [Atrás](./)