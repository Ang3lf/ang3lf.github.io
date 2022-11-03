---
title: Redundancy-Cisco
layout: default
---

*Redundancia* es la forma proporcionar a los clientes o usuarios un servicio siempre, sin interrupciones y asegurando el servicio para que los clientes "confíen" en ello, esto es posible ya que la redundancia proporciona ya sea varios equipos o múltiples rutas para el tráfico de modo que los datos puedan seguir funcionando incluso si hay una falla.

Algunos conceptos importantes antes de la configuración:

* standby: router en espera o de reserva
* active: router que se encarga de reenviar paquetes de la red
* preempt: hace que un router con mayor prioridad se convierta en router activo
* priority: es el valor que se toma para elejir a un router como activo o en reserva, los valores van de 1 a 255

Estos son algunos de los pasos que tiene el router en espera (standby):

1. EL router de reserva deja de recibir los mensajes de saludo del router que estaba reenviando los mensajes.

2. El router de reserva se convierte ahora en el router de reenvío.

3. El router de reserva toma la dirección IP y MAC virtual que se asignó, y con esto ningún host de la red siente o ve el cambio que acaban de hacer los equipos.

Existen varios protocolos de redundancia:

* *First Hop Redundancy Protocol (FHRP)*: protocolo exclusivo de Cisco y son los que proporcionan una dirección IP y MAC virtual con el fin de dar redundancia sobre el servicio de datos.

* *Host Standby Routing Protocol (HSRP)*: protocolo exclusivo de Cisco que permite la conmutación por falla, este es usado para los equipos switch de multicapa, el funcionamiento es que utiliza un grupo de routers de los cuales selecciona un router activo y otro de reserva, router activo es quien se encarga de reenviar los paquetes y el de reserva solo se encuentra en escucha hasta que falle el router activo.

* *HSRP IPv6*: tiene el mismo funcionamiento que con IPv4 solo que este procolo usa la dirección IPv6 *link-local* y MAC virtuales para asignar un router activo y en reserva

* *Virtual Redundancy Routing Protocol version 2 (VRRPv2)*: protocolo abierto que asiga de forma dinámica la responsabilidades en una red, permitiendo que los hosts de una red usen como "gateway" una dirección IP virtual que la tiene aplicada físicamente un equipo, y se elije a un router como maestro que se encarga de reenviar los paquetes y los otros router quedan como respaldo siempre teniendo en cuenta que estos equipos de respaldo distribuyan la misma IP virtual.

* *VRRPv3*: es el mismo funcionamiento que *VRRPv2* tiene la capacidad para trabajar con IPv4 e IPv6 y es más escalable que VRRPv2, creado para implementarse en redes grandes.

* *Gateway Load Balancer Protocol (GLBP)*: propiedad de Cisco que se encarga de proteger los datos contra cualquier posible falla en los routers, este protocolo hace un balanceo de cargas del tráfico entre el grupo de routers asginados asegurando que cada router siempre este funcionando adecuadamente.

* *GLBP IPv6*: permite una tolerancia a fallas en entornos de IPv6, con el grupo de routers asignados.

### Configuración

#### HSRP

Primero configuraremos el Protocolo *HSRP*:

![a1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjX7yBSl2e2ZZ2ptaltvVDY8DwmEz_zkqdEgUD1R4neJ42Q5Tr0HecrpFDKU3RU6zJW-SK5tiBa3vqbbfoVL0mZUwAdYh7MtE5qze3bTMXO7NF2LdriG-orMIAgDpLP1MiexbmNOq4IzZSYl1gYiGozPPWslBhkLMrijcxVhTYd4ReNkFeAvcBsV_ca3g/s945/Diagrama%20en%20blanco.jpeg)

```html
PE1(config)#interface Ethernet0/1
PE1(config-if)#standby 1 ip 192.168.0.1
PE1(config-if)#standby 1 name REDUNDANCY-CLIENT1
PE1(config-if)#standby 1 priority 110
PE1(config-if)#standby 1 preempt
PE1(config-if)#do show ip interface brief
Interface      IP-Address     OK?  Method      Status       Protocol 
Ethernet0/0  192.168.40.165   YES  dhcp         up            up 
Ethernet0/1   192.168.0.2     YES  unset        up            up 
Ethernet0/2   unassigned      YES  unset  administratively down down 
```
El número después de standby es el que identifca el grupo de routers para el protocolo a configurar, el rango en *HSRP* va de 0 a 4095, la prioridad 110 es el máximo por ellos es el router que se encarga del reenvío de datos, *preempt* hará que si este router se cae pero logra reestablecerse no pierda el lugar de ser el router de reenvío, la *IP virtual es 192.168.0.1* y esta misma IP deben llevar los hosts de la red, en la salida del comando de la línea 6 podemos ver que direcciones tiene este router basado en la topología y está configuración va en ambos routers a excepción de algunos parámetros:

```html
PE2(config)#interface Ethernet0/1
PE2(config-if)#standby 1 name REDUNDANCY-CLIENT1
PE2(config-if)#standby 1 ip 192.168.0.1
PE2(config-if)#standby 1 priority 90
PE2(config-if)#do show ip interface brief
Interface      IP-Address     OK?  Method      Status       Protocol 
Ethernet0/0  192.168.40.165   YES  dhcp         up            up 
Ethernet0/1   192.168.0.3     YES  unset        up            up 
Ethernet0/2   unassigned      YES  unset  administratively down down 
```
En este router también es configurado en el mismo grupo que el router 1 (R1) junto con la misma dirección *IP virtual*, la prioridad en este equipo es 90 para que quede como standby o reserva, si nos fijamos en el output del comando 'show ip interface brief' veremos que en la interfaz GigabitEthernet0/0/0 que conecta a la LAN de cliente hay un dirección IP que es diferente para ambos routers y para la *IP virtual* por lo cual en los host debemos poner que el Gateway para cada uno sea la dirección IP virutal 192.168.0.1, ahora podemos ver información sobre nuestro protocolo configurado en cada equipo, si queremos una información resumida podemos usar el comando *show ip standby brief*:
```html
PE1(config)#do show standby
Ethernet0/1 - Group 1
  State is Active
    2 state changes, last state change 00:22:29
  Virtual IP address is 192.168.0.1
  Active virtual MAC address is 0000.0c07.ac01
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.904 secs
  Preemption enabled
  Active router is local
  Standby router is 192.168.0.3, priority 90 (expires in 9.408 sec)
  Priority 110 (configured 110)
  Group name is "REDUNDANCY-CLIENT1" (cfgd)
```

```html
PE2(config)#do show standby
Ethernet0/1 - Group 1
  State is Standby
    4 state changes, last state change 00:07:16
  Virtual IP address is 192.168.0.1
  Active virtual MAC address is 0000.0c07.ac01
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.448 secs
  Preemption disabled
  Active router is 192.168.0.2, priority 110 (expires in 10.800 sec)
  Standby router is local
  Priority 90 (configured 90)
  Group name is "REDUNDANCY-CLIENT1" (cfgd)
```
Como vemos, nuestro *PE1* es el router *active* que actualmente se encarga de reenvío de paquetes ya que cuenta con la prioridad más alta. También usamos como switch un equipo *Mikrotik* este equipo solo se encarga de actuar en capa 2, las configuraciones hechas en este mikrotik, este equipo es necesario para que los routers puedan tener comunicación entre sí y no dependa uno del otro para comunicarse, ya que un switch puede hacer esto:
```html
[admin@CE] > interface bridge port add bridge=client1 interface=ether1-ether5
[admin@CE] > int bridge port print
Flags: X - disabled, I - inactive, D - dynamic, H - hw-offload
 #     INTERFACE     BRIDGE        HW  PVID PR  PATH-COST INTERNA...    HORIZON
 0     ether1        client1       yes    1 0x         10         10       none
 1     ether2        client1       yes    1 0x         10         10       none
 2     ether5        client1       yes    1 0x         10         10       none
 3     ether3        client1       yes    1 0x         10         10       none
 4     ether4        client1       yes    1 0x         10         10       none
```
Ahora podemos probar en un equipo cliente conexión a internet, en este equipo cliente es necesario que como Gateway tenga la dirección *IP virtual*

![a2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjgcqt1toBH2NRdcvqjpfMhTn_wSPW_26WZvNQpDl_nZoJEZKLf6v7fq3Ahn3mk0Z3JcBvLBdzg9padMzQWPTRWq5jHrfScW3M7-wYCpkEkYgryw_ZOAtbMu5BtDp-4-ximQnR8dEeuDoPzMlo2OtGttC7v4-0D5shb_VGCAQIuZOEGPTpcwGf8WkYt8w/s640/kali.JPG)

Tenemos servicio, hay salida a internet, pero para comprobar nuestras configuraciones, apagamos el *PE1* y veremos como cambia el *PE2* y luego de unos segundos el equipo cliente tendra comunicación y salida a internet como si nada hubiera pasado
```html
PE2(config)#do show standby
Ethernet0/1 - Group 1
  State is Active
    5 state changes, last state change 00:00:26
  Virtual IP address is 192.168.0.1
  Active virtual MAC address is 0000.0c07.ac01
    Local virtual MAC address is 0000.0c07.ac01 (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.840 secs
  Preemption disabled
  Active router is local
  Standby router is unknown
  Priority 90 (configured 90)
  Group name is "REDUNDANCY-CLIENT1" (cfgd)
```
Ahora el *PE2* se ha convertido en el router *active* o de reenvío y podemos ver el tiempo que lleva desde cambió de estado, ahora como tema de diagnóstico podemos ver si tenemos acceso al *mikrotik* si hay algún equipo que estaba activo por medio de la direcciones MAC aprendidas y ahora ya no y ver el tiempo y reestablecerlo
```html
[admin@CE] > int bridge host print
Flags: X - disabled, I - invalid, D - dynamic, L - local, E - external
 #       MAC-ADDRESS        VID ON-INTERFACE     BRIDGE    AGE
 0   D   00:00:0C:07:AC:01      ether2           client1   1s
 1   D   00:50:00:00:05:00      ether5           client1   2m25s
 2   DL  50:00:00:03:00:00      ether1           client1
 3   DL  50:00:00:03:00:01      ether2           client1
 4   DL  50:00:00:03:00:02      ether3           client1
 5   DL  50:00:00:03:00:03      ether4           client1
 6   DL  50:00:00:03:00:04      ether5           client1
 7   D   AA:BB:CC:00:20:10      ether2           client1   7s
```

También podemos agregar más configuraciones a nuestro *HSRP*, si necesitamos configurar diferentes tiempos de espera y saludo, de demora, autenticación entre otros:
```html
  standby 1 timers 5 15
  standby 1 preempt delay minimum 300
  standby mac-refresh 30
```
Estas son configuraciones en cuanto a configuraciones del tiempo para manejo de la redundancia local, es importante que si modificamos los tiempos que trae por deafult debemos recordar que también debe ser modificado en el otro equipo, es decir ambos equipos (routers) deben tener los mismos tiempos de *hello* y *hold*.

Con *HSRP* podemos grupos distintos del protocolo para poder tener un balanceo de carga en la red y así logramos que un solo equipo no sea saturado, por ejemplo, tomemos en cuenta la misma topología:

```html
PE1(config)# interface Ethernet 0/1
PE1(config-if)# ip address 192.168.0.3 255.255.255.0
PE1(config-if)# standby 1 priority 110
PE1(config-if)# standby 1 preempt
PE1(config-if)# standby 1 ip 192.168.0.1
PE1(config-if)# standby 2 preempt
PE1(config-if)# standby 2 ip 192.168.0.2
```

```html
PE2(config)# interface Ethernet 0/1
PE2(config-if)# ip address 192.168.0.4 255.255.255.0
PE2(config-if)# standby 1 preempt
PE2(config-if)# standby 1 192.168.0.1
PE2(config-if)# standby 2 priority 110
PE2(config-if)# standby 2 preempt
PE2(config-if)# standby 2 ip 192.168.0.2
```
Con estas configuraciones hechas en cada router, tendremos un balanceo de cargas de la red ya que si tenemos 4 hosts en la red, podemos poner 2 ellos (host) que se peguen al router *PE1* y lo 2 restantes (hosts) al router *PE2* y teniendo 2 grupos configurados en cada router aseguramos que si cualquier router falla el otro tenga toda la carga del servicio y viceversa, ahora tenemos otra forma de configurar grupos para *HSRP* que una forma más rápida ya que en este solo necesitamos que lleve la dirección *IP virtual* y con el comando *standby 5 follow name_hsrp* haremos que el equipo también sea tomado como cliente para el protocolo.

```html
PE1(config-if)# standby 1 ip 192.168.0.1
PE1(config-if)# standby 1 priority 110
PE1(config-if)# standby 1 preempt
PE1(config-if)# standby 1 name HSRP_TEST
```

```html
PE2(config-if)# standby 2 ip 10.0.0.254
PE2(config-if)# standby 2 follow HSRP1  
```

Para hacer que nuestro protocolo *HSRP* funcione con autenticación, primero debemos crear una llave de autenticación en el router para el grupo:

```html
PE1(config-if)# standby 1 priority 100
PE1(config-if)# standby 1 preempt
PE1(config-if)# standby 1 ip 192.168.0.1
PE1(config-if)# standy 1 authentication md5 key-string C!sc0123
```

```html
PE2(config-if)# standby 1 priority 100 -> (default)
PE2(config-if)# standby 1 ip 192.168.0.1
PE2(config-if)# standy 1 authentication md5 key-string C!sc0123
```

#### VRRP

Protocolo libre, es posible configurarlo en equipos de diferentes proveedores, para ello lo configuramos en un equipo *Huawei* y uno *Cisco*, siempre usando la misma topología del protocolo anterior *HSRP*:

![a3](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhACKNR64LuY3gQrJI0eBVwVSQZ__if5dCRQOpWcEP2Cy9EZIuTIS8qs1KEGit_kurVG87cICpduKfklwtDkJHx-OnpzAMnja1hUygeSCqAiftU3E4JgcLnCd8IKBN0d3xT9DNQgtMNAbxtbhdPOSI_1Ag25PCcCXWnoVux7qfD6f8pWQuwbsJxUhgjrg/s945/Diagrama%20en%20blanco.jpeg)

```html
[~PE1-Ethernet1/0/1]disp this
#
interface Ethernet1/0/1
 description Customer1
 undo shutdown
 ip address 192.168.0.2 255.255.255.224
 vrrp vrid 1 virtual-ip 192.168.0.1
 vrrp vrid 1 priority 110
 vrrp vrid 1 preempt-mode disable
#
return
[~PE1-Ethernet1/0/1]
```
```html
PE2(config)#int ethe0/1
PE2(config-if)#vrrp 1 ip 192.168.0.1
PE2(config-if)#vrrp 1 priority 150
PE2(config-if)#vrrp 1 description Customer-1
```
En este caso por default el router Huawei trae configurado *preempt* por lo que si necesitamos deshabilitarlo para que este no vuelva a ser el master luego de una interrupción debemos poner: *vrrp vrid 1 preempt-mode disable* y en los equipos Cisco es: *vrrp 1 preempt*, ahora veamos el estado de los equipos:
```html
[~PE1]disp vrrp
Ethernet1/0/1 | Virtual Router 1
State          : Master
Virtual IP     : 192.168.0.1
Master IP      : 192.168.0.2
PriorityRun    : 150
PriorityConfig : 150
MasterPriority : 150
Preempt        : YES   Delay Time : 0s
Hold Multiplier: 3
TimerRun       : 1s
TimerConfig    : 1s
Auth Type      : NONE
Virtual MAC    : 0000-5e00-0101
Check TTL      : YES
Config Type    : normal-vrrp
Backup-forward : disabled
Create Time       : 2022-11-02 23:01:40
Last Change Time  : 2022-11-02 23:01:47
```

```html
PE2#show vrrp
Ethernet0/1 - Group 1
Customer-1
  State is Master
  Virtual IP address is 192.168.0.1
  Virtual MAC address is 0000.5e00.0101
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 150
  Master Router is 192.168.0.3 (local), priority is 150
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.414 sec
```

El equipo Huawei está como *backup*, mientras que *PE2* está como router *master* que se encarga del reenvío de paquetes de la red, también podemos ver que ambos equipos tienen las mismas *direcciones MAC e IP virtuales* y esto es porque el protocolo como tal lo indica, si necesitamos ver la información más resumida podemos usar los comandos:

```html
[~PE1]disp vrrp 1 brief
Total:1     Master:0    Backup:1    Non-active:0
VRID State       Interface               Type    Virtual IP
----------------------------------------------------------------
1    Backup      Eth1/0/1                Normal  192.168.0.1

PE2#show vrrp brief
Interface          Grp Pri Time  Own Pre State   Master addr     Group addr
Et0/1              1   150 3414       Y  Master  192.168.0.3     192.168.0.1
```
Ahora que sabemos que nuestro *VRRP* está configurado podemos probar apagar la interfaz de un equipo o apagarlo directamente, y comprobar que los clientes en la LAN puedan seguir teniendo salida a internet sin ningun problema

![a4](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhkvchUvQ4RLMUzVkHGuFNbs5PlUpVfnLG3eMBseYDUokx4tC1vO4QwNdeb6tC4B4mJ5UB6ftHaU1pP6q5YLnNNaqvDNjj5ZyFmS19ekmYxhVbjIKVPvjusqyWIJN4CujfkNkFlTJhdve6-2bzGvnWuFRMx4HBG0emJeZ9xwaE7VBKltdgqJgy4Rh9eUA/s598/windows.JPG)

![a5](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhVRXIMK_QRokmYx1b4syuyVa-vhpG7sf2fEScuLz8nYkShAdGzz45n3UxELQg0bfHpFmuVBP939kV1K5C-eRlot2u5GtxM-5SEDeKf92AWgduPzDPooPpD8qOO5cLl44PNEeGOHpIELAnF73mRJ0ChsqcMfQ3ymKXKmC6idveftanwBYDGrH457DQciw/s859/kali1.JPG)

En Huawei podemos ver como hace todo este cambio de backup a master cuando deja de recibir mensajes del *PE2*
```html
[~PE1]disp vrrp state-change interface Ethernet 1/0/1 vrid 1
Time                            SourceState    DestinationState      Reason
-----------------------------------------------------------------------------
2022-11-03 00:14:15             Master         Backup                Priority calculation
2022-11-03 00:11:03             Backup         Master                Protocol timer expired
2022-11-03 00:00:49             Master         Backup                Priority calculation
2022-11-03 00:00:46             Backup         Master                Protocol timer expired
2022-11-03 00:00:42             Initialize     Backup                Interface up

[~PE1]disp vrrp
Ethernet1/0/1 | Virtual Router 1
State          : Master
Virtual IP     : 192.168.0.1
Master IP      : 192.168.0.2
PriorityRun    : 110
PriorityConfig : 110
MasterPriority : 110
Preempt        : NO
Hold Multiplier: 3
TimerRun       : 1s
TimerConfig    : 1s
Auth Type      : NONE
Virtual MAC    : 0000-5e00-0101
Check TTL      : YES
Config Type    : normal-vrrp
Backup-forward : disabled
Create Time       : 2022-11-03 00:00:39
Last Change Time  : 2022-11-03 00:11:03
```
Podemos ver el intercambio de estados que hace y como se convierte en router *master* mientras el *PE2* esté sin funcionamiento, el comando usado en Huawei nos sirve bastante para ver en que momento cambia de estado, es los equipos Cisco podemos usar *show vrrp interface ethernet 0/1 group 1* pero esto da una información similar a la del comando simple *show vrrp*

Para configurar *VRRPv3*:
```html
[~PE1]vrrp version v3
[*PE1]vrrp virtual-ip ping enable
[*PE1]int ethe1/0/1
[*PE1-Ethernet1/0/1]ipv6 enable
[*PE1-Ethernet1/0/1]ipv6 address 2001:db8:acad:1::1/64
[*PE1-Ethernet1/0/1]vrrp vrid 1 version-3 send-packet-mode v2v3-both
[*PE1-Ethernet1/0/1]vrrp6 vrid 1 virtual-ip fe80::11:1 link-local
[*PE1-Ethernet1/0/1]vrrp6 vrid 1 priority 150
[*PE1-Ethernet1/0/1]commit
[~PE1]display vrrp6 brief
Total:1     Master:0    Backup:0    Non-active:1
VRID State       Interface               Type    Virtual IP
----------------------------------------------------------------
1    Initialize  Eth1/0/1                Normal  FE80::11:1
[~PE1]display vrrp6 1
                    ^
Error: Too many parameters found at '^' position.
[~PE1]display vrrp6 vri
[~PE1]display vrrp6 vrid 1
Ethernet1/0/1 | Virtual Router 1
State           : Master
Virtual IP      : FE80::11:1
Master IP       : FE80::3A64:83FF:FEBB:C901
PriorityRun     : 150
PriorityConfig  : 150
MasterPriority  : 150
Preempt         : YES   Delay Time : 0s
Hold Multiplier : 3
TimerRun        : 100cs
TimerConfig     : 100cs
Virtual MAC     : 0000-5e00-0201
Check hop limit : YES
Config Type     : normal-vrrp
Backup-forward  : disabled
Create Time       : 2022-11-03 16:55:23
Last Change Time  : 2022-11-03 16:55:30
```

```html
PE2(config)#fhrp version vrrp v3
PE2(config)# interface vlan 11
PE2(config-if)# vrrp 11 address-family ipv4
PE2(config-if-vrrp)# address 192.168.0.254
PE2(config-if-vrrp)# priority 150
PE2(config-if-vrrp)# exit
PE2(config-if)# vrrp 11 address-family ipv6
PE2(config-if-vrrp)# address fe80::11:1 primary
PE2(config-if-vrrp)# priority 150
PE2(config-if-vrrp)# exit
PE2(config)#ipv6 unicast-routing
```

La autenticación en este protocolo es similar a *HSRP* y el cambio de tiempos de *hello* y *hold* también, importante que ambos tengan los mismoa valores para no tener errores

Huawei:

```html
vrrp vrid 1 authentication-mode md5 Cisco123
```

Cisco:
```html
vrrp 1 authentication md5 key-string Cisco123
```

#### GLBP

Usaremos la misma topología.

![a6](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhACKNR64LuY3gQrJI0eBVwVSQZ__if5dCRQOpWcEP2Cy9EZIuTIS8qs1KEGit_kurVG87cICpduKfklwtDkJHx-OnpzAMnja1hUygeSCqAiftU3E4JgcLnCd8IKBN0d3xT9DNQgtMNAbxtbhdPOSI_1Ag25PCcCXWnoVux7qfD6f8pWQuwbsJxUhgjrg/s945/Diagrama%20en%20blanco.jpeg)

Antes de configurarlo debemos saber algunos conceptos sobre este protocolo:

* *Active Virtual Gateway (AVG)*: es el router encargado dentro del proceso de redundancia de hacer las resoluciones ARP que le soliciten, en esta topolgía diremos que nuestro *AVG* es *PE1*

* *Active Virtual Forwarding (AVF)*: es o son los routers encargados del reenvío de los paquetes de toda la red, primero sabemos que *GLBP* es un otro protocolo de redundancia pero a diferencia de *HSRP* o *VRRP* este protocolo es capaz de hacer un equilibrio de carga en tipo <a href="https://ang3lf.github.io/1-1-lb.html" target="_blank">*Round Robin*</a> que básicamente atiende peticiones uno cada uno por "turnos", así que en esta topología diremos que nuestro *AVF* son los dos routers *PE1* y *PE2* ya que ambos atenderán los paquetes que le lleguen.

```html
PE1(config)#do show ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.40.165  YES DHCP   up                    up  
Ethernet0/1                192.168.0.1     YES manual up                    up  
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
PE1(config)#do show ipv6 int brief
Ethernet0/0            [up/up]
    unassigned
Ethernet0/1            [up/up]
    FE80::1:1
    2001:DB8:ACAD:1::1
Ethernet0/2            [administratively down/down]
    unassigned
Ethernet0/3            [administratively down/down]
    unassigned
PE1(config)#ipv6 unicast-routing
PE1(config)#int ethe0/1
PE1(config-if)#glbp 5 ip 192.168.0.30
PE1(config-if)#glbp 5 preempt
PE1(config-if)#glbp 5 priority 170
PE1(config-if)#glbp 5 authen md5 key-string C!sc0123
PE1(config-if)#glbp 55 ipv6 fe80::1:254
PE1(config-if)#glbp 55 priority 170
PE1(config-if)#glbp 55 preempt
PE1(config-if)#glbp 55 authentication md5 key-string C!sc0v6
```
```html
PE2(config)#do show ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                192.168.40.164  YES DHCP   up                    up  
Ethernet0/1                192.168.0.2     YES manual up                    up  
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
PE2(config)#do show ipv6 int brief
Ethernet0/0            [up/up]
    unassigned
Ethernet0/1            [up/up]
    FE80::1:2
    2001:DB8:ACAD:1::2
Ethernet0/2            [administratively down/down]
    unassigned
Ethernet0/3            [administratively down/down]
    unassigned
PE2(config)#ipv6 unicast-routing
PE2(config)#int ethe0/1
PE2(config-if)#glbp 5 ip 192.168.0.30
PE2(config-if)#glbp 5 preempt
PE2(config-if)#glbp 5 authentication md5 key-string C!sc0123
PE2(config-if)#glbp 55 ipv6 fe80::1:254
PE2(config-if)#glbp 55 preempt
PE2(config-if)#glbp 55 authentication md5 key-string C!sc0v6
```

Comprobamos las configuraciones en ambos PE´s:
```html
PE1(config)#do show glbp brief
Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Et0/1       5    -   170 Active   192.168.0.30    local           192.168.0.2
Et0/1       5    1   -   Active   0007.b400.0501  local           -
Et0/1       5    2   -   Listen   0007.b400.0502  192.168.0.2     -
Et0/1       55   -   170 Active   FE80::1:254     local           FE80::1:2
Et0/1       55   1   -   Listen   0007.b400.3701  FE80::1:2       -
Et0/1       55   2   -   Active   0007.b400.3702  local           -
```

```html
PE2(config)#do show glbp brief
Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Et0/1       5    -   100 Standby  192.168.0.30    192.168.0.1     local
Et0/1       5    1   -   Listen   0007.b400.0501  192.168.0.1     -
Et0/1       5    2   -   Active   0007.b400.0502  local           -
Et0/1       55   -   100 Standby  FE80::1:254     FE80::1:1       local
Et0/1       55   1   -   Active   0007.b400.3701  local           -
Et0/1       55   2   -   Listen   0007.b400.3702  FE80::1:1       -
```
Observamos que ya ambos routers se conocen y están listos ante cualquier posible falla en la red, pero veamos en los clientes su comportamiento, recordemos la dirección *MAC Virtual* que no está dando *GLBP* que es *0007.b400.05* la última parte varia de la MAC porque cada router tiene una MAC virtual diferente

![a7](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiL-p3GAFI4qJEE2k1KqH7hbqGpvbREZP3kH-Fxp2dVwNTKOWSWokzycnRh3kSJ7Cn-Ak6O9xcc7x4_bR6ZlRhsxiiY1WkwMkcXkdadLFnNxWmzLV1Oc_sShuf6zJmE8q335kGxk-BLo4UPUTWAnjN_THt75nHAPR79yCkLeHTlu8wcDEeef-Lll8P3kg/s604/win-arp.JPG)

![a8](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjOBjeKymuG5a1xxYOSfTAz3m_yul2btSkA80h7jrR3srhMNkDqlmMcofKvEYsMym9X7fiBegnJe6uHRaDl0-x2_dBJyqW7exSxqHKCtcbLh8hlRRPxAK_54ar3eNpMQ2mE-s4J95Fm7IXoMdHsYK2hxJyV-Inpt_THCrSU_ibEkY5iJs911NgUTtxZQg/s638/win-arp6.JPG)

![a9](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiGsB8SQKC3F0wbpOZW8POJUp92tjeTX-F7Jip2qTKXHcZSL8k09T7XRDWOwu5TDILPpgqTdxSRG2wwXwDeyIaejvzE9bdbq330c7mGuBjN8_HkhFoizBmkmpAS3N_Tzm-UEGpa59RrQSNsRwctSEUe4jpzJgeI67BjOmpAwJu7DuDMjd3W_KxcJz-66Q/s631/jali-arp.JPG)

El equipo Windows tiene la dirección *MAC 0007.b400.0501* como vecino, y es la MAC virtual de nuestro *AVG* *PE1*, y el equipo Linux tiene la *MAC 0007.b400.0502* que es correspondiente al *AVF* *PE2*, básicamente con esto determinamos que el protocolo *GLBP* usa el algoritmo de <a href="https://ang3lf.github.io/1-1-lb.html" target="_blank">*Round Robin*</a> para hacer el balanceo de cargas, este protocolo es explicado también en otros posts de esta página, para comprobar correctamente se generó tráfico al mismo tiempo en ambos clientes *Windows* y *Linux*. Ojo que también podríamos ver que dispositivo recibe el paquete usando *traceroute* pero con ambos comandos veremos el mismo resultado.

```html
PE1#show glbp
Ethernet0/1 - Group 5
  State is Active
    5 state changes, last state change 00:32:03
  Virtual IP address is 192.168.0.30
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.256 secs
  Redirect time 600 sec, forwarder timeout 14400 sec
  Authentication MD5, key-string
  Preemption enabled, min delay 0 sec
  Active is local
  Standby is 192.168.0.2, priority 100 (expires in 9.824 sec)
  Priority 170 (configured)
  Weighting 100 (default 100), thresholds: lower 1, upper 100
  Load balancing: round-robin
  Group members:
    aabb.cc00.1010 (192.168.0.1) local
    aabb.cc00.2010 (192.168.0.2) authenticated
  There are 2 forwarders (1 active)
  Forwarder 1
    State is Active
      5 state changes, last state change 00:32:03
    MAC address is 0007.b400.0501 (default)
    Owner ID is aabb.cc00.1010
    Redirection enabled
    Preemption enabled, min delay 30 sec
    Active is local, weighting 100
    Client selection count: 8
  Forwarder 2
    State is Listen
    MAC address is 0007.b400.0502 (learnt)
    Owner ID is aabb.cc00.2010
    Redirection enabled, 599.840 sec remaining (maximum 600 sec)
    Time to live: 14399.840 sec (maximum 14400 sec)
    Preemption enabled, min delay 30 sec
    Active is 192.168.0.2 (primary), weighting 100 (expires in 10.048 sec)
    Client selection count: 8
Ethernet0/1 - Group 55
  State is Active
    1 state change, last state change 00:36:49
  Virtual IP address is FE80::1:254
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.496 secs
  Redirect time 600 sec, forwarder timeout 14400 sec
  Authentication MD5, key-string
  Preemption enabled, min delay 0 sec
  Active is local
  Standby is FE80::1:2, priority 100 (expires in 8.448 sec)
  Priority 170 (configured)
  Weighting 100 (default 100), thresholds: lower 1, upper 100
  Load balancing: round-robin
  Group members:
    aabb.cc00.1010 (FE80::1:1) local
    aabb.cc00.2010 (FE80::1:2) authenticated
  There are 2 forwarders (1 active)
  Forwarder 1
    State is Listen
    MAC address is 0007.b400.3701 (learnt)
    Owner ID is aabb.cc00.2010
    Redirection enabled, 598.464 sec remaining (maximum 600 sec)
    Time to live: 14398.464 sec (maximum 14400 sec)
    Preemption enabled, min delay 30 sec
    Active is FE80::1:2 (primary), weighting 100 (expires in 10.240 sec)
    Client selection count: 1
  Forwarder 2
    State is Active
      1 state change, last state change 00:36:35
    MAC address is 0007.b400.3702 (default)
    Owner ID is aabb.cc00.1010
    Redirection enabled
    Preemption enabled, min delay 30 sec
    Active is local, weighting 100
    Client selection count: 1
```
```html
PE2#show glbp
Ethernet0/1 - Group 5
  State is Standby
    1 state change, last state change 00:31:47
  Virtual IP address is 192.168.0.30
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.592 secs
  Redirect time 600 sec, forwarder timeout 14400 sec
  Authentication MD5, key-string
  Preemption enabled, min delay 0 sec
  Active is 192.168.0.1, priority 170 (expires in 8.160 sec)
  Standby is local
  Priority 100 (default)
  Weighting 100 (default 100), thresholds: lower 1, upper 100
  Load balancing: round-robin
  Group members:
    aabb.cc00.1010 (192.168.0.1) authenticated
    aabb.cc00.2010 (192.168.0.2) local
  There are 2 forwarders (1 active)
  Forwarder 1
    State is Listen
    MAC address is 0007.b400.0501 (learnt)
    Owner ID is aabb.cc00.1010
    Time to live: 14397.344 sec (maximum 14400 sec)
    Preemption enabled, min delay 30 sec
    Active is 192.168.0.1 (primary), weighting 100 (expires in 7.424 sec)
  Forwarder 2
    State is Active
      1 state change, last state change 00:31:50
    MAC address is 0007.b400.0502 (default)
    Owner ID is aabb.cc00.2010
    Preemption enabled, min delay 30 sec
    Active is local, weighting 100
Ethernet0/1 - Group 55
  State is Standby
    3 state changes, last state change 00:36:45
  Virtual IP address is FE80::1:254
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.352 secs
  Redirect time 600 sec, forwarder timeout 14400 sec
  Authentication MD5, key-string
  Preemption enabled, min delay 0 sec
  Active is FE80::1:1, priority 170 (expires in 9.632 sec)
  Standby is local
  Priority 100 (default)
  Weighting 100 (default 100), thresholds: lower 1, upper 100
  Load balancing: round-robin
  Group members:
    aabb.cc00.1010 (FE80::1:1) authenticated
    aabb.cc00.2010 (FE80::1:2) local
  There are 2 forwarders (1 active)
  Forwarder 1
    State is Active
      1 state change, last state change 00:36:46
    MAC address is 0007.b400.3701 (default)
    Owner ID is aabb.cc00.2010
    Preemption enabled, min delay 30 sec
    Active is local, weighting 100
  Forwarder 2
    State is Listen
    MAC address is 0007.b400.3702 (learnt)
    Owner ID is aabb.cc00.1010
    Time to live: 14399.424 sec (maximum 14400 sec)
    Preemption enabled, min delay 30 sec
    Active is FE80::1:1 (primary), weighting 100 (expires in 10.816 sec)
```
Aqui vemos más detalles de la configuración realizada para ambos protocolos *IPv4* e *IPv6*, se manejan tiempos de espera y saludo iguales, nos muestra la dirección *MAC* que tiene cada interfaz en ambos routers y la guarda en su tabla, muestra también la MAC virtual que brinda para ambos equipos, también vemos que en ambos outputs de los routers tiene la sección de *Forwarder number-id* y esto es porque como se explicó al inicio de este protocolo ambos equipos pueden reenviar paquetes que reciban con la finalidad de que ni uno de estos se vea saturado.

Con GLBP se definen dos umbrales: un umbral inferior que se aplica cuando la *AVF* pierde peso y un umbral superior que se aplica cuando la *AVF* recupera el peso. El mecanismo de GLBP tiene dos umbrales que son estos, *superior* e *inferior* definidos. Esto ofrece más flexibilidad que sus contrapartes HSRP y VRRP, que solo permiten definir un único umbral. Si la prioridad o peso del *AVF* cae por debajo del umbral, *AVF* pierde su estado *activo* y tan pronto como el peso o la prioridad del *AVF* supere el umbral superior, este *AVF* recupera su estado activo, para ello necesitamos hacer un seguimiento (track) a una interfaz y ver que los valores sean ajustados nuevamente, la interfaz para el seguimiento debe ser la conecte a otra red o de salida a WAN ya que así conseguiremos un buen ajuste de peso en el equipo

```html
PE1(config)#track 5 interface ethernet 0/0 line-protocol
PE1(config-track)#exit
PE1(config)#int ethe0/1
PE1(config-if)#glbp 5 weighting 110 lower 85 upper 105
PE1(config-if)#glbp 5 weighting track 5 decrement 30
PE1(config-if)#glbp 55 weighting 110 lower 85 upper 105
PE1(config-if)#glbp 55 weighting track 5 decrement 30
```

Se configura con un peso glbp de 110, un umbral inferior de 85 y un umbral superior de 105, cuando el peso cae por debajo del umbral inferior especificado (85), el *PE1* *AVF* se ve obligado a renunciar a su función para la dirección *MAC Active* que se le ha asignado, en el segundo comando, *GLBP* se asocia con el estado del protocolo de la interfaz *Ethernet 0/0*. Si el estado del protocolo de línea de esta interfaz cambia, el peso configurado para 110 se reducirá en 30, lo que dará como resultado un peso de 80 y *PE1* perdería entonces su función *AVF* hasta que el peso supere el umbral superior definido de 105, son los mismos comandos para el grupo 55 y los cuatro comandos se aplican a la interfaz *Ethernet 0/1* en el interruptor *PE1*.

```html
PE1(config)#int ethe0/0
PE1(config-if)#shut
PE1(config-if)#do show glbp
Ethernet0/1 - Group 5
  State is Active
    1 state change, last state change 01:11:39
  Virtual IP address is 192.168.0.30
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.248 secs
  Redirect time 600 sec, forwarder timeout 14400 sec
  Authentication MD5, key-string
  Preemption enabled, min delay 0 sec
  Active is local
  Standby is 192.168.0.2, priority 100 (expires in 7.584 sec)
  Priority 170 (configured)
  Weighting 80, low (configured 110), thresholds: lower 85, upper 105
    Track object 5 state Down decrement 30
  Load balancing: round-robin
  Group members:
    aabb.cc00.1010 (192.168.0.1) local
    aabb.cc00.2010 (192.168.0.2) authenticated
  There are 2 forwarders (1 active)
  Forwarder 1
    State is Listen
      4 state changes, last state change 01:04:55
    MAC address is 0007.b400.0501 (learnt)
    Owner ID is aabb.cc00.2010
    Redirection enabled, 597.600 sec remaining (maximum 600 sec)
    Time to live: 14397.600 sec (maximum 14400 sec)
    Preemption enabled, min delay 30 sec
    Active is 192.168.0.2 (primary), weighting 100 (expires in 9.312 sec)
    Client selection count: 4
  Forwarder 2
    State is Active
      1 state change, last state change 01:04:53
    MAC address is 0007.b400.0502 (default)
    Owner ID is aabb.cc00.1010
    Redirection enabled
    Preemption enabled, min delay 30 sec
    Active is local, weighting 80
    Client selection count: 4
Ethernet0/1 - Group 55
  State is Active
    1 state change, last state change 01:11:38
  Virtual IP address is FE80::1:254
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 0.320 secs
  Redirect time 600 sec, forwarder timeout 14400 sec
  Authentication MD5, key-string
  Preemption enabled, min delay 0 sec
  Active is local
  Standby is FE80::1:2, priority 100 (expires in 9.504 sec)
  Priority 170 (configured)
  Weighting 80, low (configured 110), thresholds: lower 85, upper 105
    Track object 5 state Down decrement 30
  Load balancing: round-robin
  Group members:
    aabb.cc00.1010 (FE80::1:1) local
    aabb.cc00.2010 (FE80::1:2) authenticated
  There are 2 forwarders (1 active)
  Forwarder 1
    State is Listen
      2 state changes, last state change 01:05:35
    MAC address is 0007.b400.3701 (learnt)
    Owner ID is aabb.cc00.2010
    Redirection enabled, 599.520 sec remaining (maximum 600 sec)
    Time to live: 14399.520 sec (maximum 14400 sec)
    Preemption enabled, min delay 30 sec
    Active is FE80::1:2 (primary), weighting 100 (expires in 9.536 sec)
    Client selection count: 1
  Forwarder 2
    State is Active
      1 state change, last state change 01:05:22
    MAC address is 0007.b400.3702 (default)
    Owner ID is aabb.cc00.1010
    Redirection enabled
    Preemption enabled, min delay 30 sec
    Active is local, weighting 80
```
Si apagamos la interfaz del *track* veremos que tal como se explicó al principio el valor o el peso ahora es de 80 en *PE1* y los estados de este mismo equipo han cambiado, también podemos ver estadísitcas de la interfaz ahora

```html
PE1(config-if)#do show glbp brief
Interface   Grp  Fwd Pri State    Address         Active router   Standby router
Et0/1       5    -   170 Active   192.168.0.30    local           192.168.0.2
Et0/1       5    1   -   Listen   0007.b400.0501  192.168.0.2     -
Et0/1       5    2   -   Listen   0007.b400.0502  192.168.0.2     -
Et0/1       55   -   170 Active   FE80::1:254     local           FE80::1:2
Et0/1       55   1   -   Listen   0007.b400.3701  FE80::1:2       -
Et0/1       55   2   -   Listen   0007.b400.3702  FE80::1:2       -

PE1(config)#do show interface ethe0/1 stats
Ethernet0/1
          Switching path    Pkts In   Chars In   Pkts Out  Chars Out
               Processor       5455    1080039       4644     681412
             Route cache        201      20597         23       2713
                   Total       5656    1100636       4667     684125
```

Ya con esto también configuramos otro algoritmo de balanceo de cargas, recordemos que este protocolo por default usa <a href="https://ang3lf.github.io/1-1-lb.html" target="_blank">*Round Robin*</a>.

## [Atrás](./index.md)