---
title: Switching
layout: default
---
*Switch Vlans* es un método de routing que puede ser con vlans diferentes o vlans iguales que pasan por un enlace truncal, una Vlan divide dominios de difusión en un entorno LAN pero cuando los hosts de esa LAN quieren comunicarse con hosts de otra Vlan necesitamos que el tráfico sea enrutable, y aquí es donde entra el routing entre vlans.

### Configuración

Iremos de lo más básico a lo 'complicado' o largo, lo primero será configurar switches truncales:

![toposwitch](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj2GM7uwWeQ3LygSSlyZkp_4cRlis2wMMEMEZBqeonAPTbOkQMBXocQW87E94BYsVF-zCwpvvZc0a9iPV7tlk2Fo77VDDRMTg4AOFQSTKgaz27HcB_xMOf2jAaWPQlrLIhCv76grSbPTOaMX10II3tUfW1y51IEbK21HIdCd8idGtQU07njyLbj5bURAg/s525/topology.jpeg)

En cada switch se configuran 5 vlans, 3 son para datos, 1 de administración y la última es la nativa

* *Datos:* Es donde pasa tdoo el tráfico que etiquetemos.
* *Administración:* Es la vlan usada para comunicar a los dispositivos de switch a switch o de router a switch, entre otros.
* *Nativa:* Usada para tráfico sin etiquetar cuando el puerto esta en modo trunk 802.1Q

```html
S1(config)#vlan 5
S1(config-vlan)#name DATOS-1
S1(config-vlan)#vlan 10
S1(config-vlan)#name DATOS-2
S1(config-vlan)#vlan 15
S1(config-vlan)#name DATOS-3
S1(config-vlan)#vlan 20
S1(config-vlan)#name IT
S1(config-vlan)#vlan 25
S1(config-vlan)#name NATIVA
S1(config-vlan)#exit
S1(config-vlan)#interface range fastEthernet 0/1-5
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 5
S1(config-if-range)#interface range f0/6-11
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 10
S1(config-if-range)#interface range f0/12-16
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 15
S1(config-if-range)#exit
S1(config)#interface vlan 20
S1(config-if)#ip address 10.179.1.2 255.255.255.252
S1(config-if)#interface fastEthernet 0/24
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 25
S1(config-if)#switchport nonegotiate
```
Primero se crean las vlans con las que esteremos trabajando, luego les asignamos las interfaces con las que trabajará cada vlan (el rango de puertos), la única vlan que puede llevar direccionamiento en los switches es la vlan20 que es la vlan de administración, por último en la interfaz FastEthernet0/24 como es la que está conectada al otro switch debemos poner para que quede configurada como modo truncal así permitir el envío y recepción de paquetes de distintos puntos, asignamos que la vlan nativa sea la 25, 'switchport nonegotiate' evita que los paquetes de negocicación se envíen por la interfaz configurada.

> Estas configuraciones deben ser iguales en el 'S2', solo debemos poner la ip correspondiente.

### Multilayer switch

Haremos configuraciones de routing con un switch capa 3

![multilayer](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgDr_UvoeewPlBaQLItft60hBFRwGYB07S0t5VfWwEuaelqUiV8pXpUXTVOhYJhswJDJ1OoBXOK7KaTKYL-L336N1I-3aIy6obG9Ji38Cv78AkKjb9J3MOOEX90Fo42345a4iAbe8yTlXAA6_aBgVuIzwrKlg6mSoROk2xyaseWhuQKQvdkESnkcjAJOw/s826/intervlan.jpeg)

```html
SLayer3(config)#ip routing
SLayer3(config)#ipv6 unicast-routing
SLayer3(config)#sdm prefer dual-ipv4-and-ipv6 default
SLayer3(config)#vlan 5
SLayer3(config-vlan)#name Data-1
SLayer3(config-vlan)#vlan 10
SLayer3(config-vlan)#name Data-2
SLayer3(config-vlan)#vlan 15
SLayer3(config-vlan)#name VoIP
SLayer3(config-vlan)#vlan 20
SLayer3(config-vlan)#name Admin
SLayer3(config-vlan)#vlan 25
SLayer3(config-vlan)#name Native
SLayer3(config-vlan)#interface vlan 5
SLayer3(config-if)#ip address 192.168.0.1 255.255.255.0
SLayer3(config-if)#ipv6 address 2001:db8:acad:a::1/64
SLayer3(config-if)#ipv5 address fe80::1 link-local
SLayer3(config-if)#description VLAN DATOS-1
SLayer3(config-if)#interface vlan 10
SLayer3(config-if)#ip address 172.16.0.1 255.255.0.0
SLayer3(config-if)#ipv6 address 2001:db8:acad:b::1/64
SLayer3(config-if)#ipv6 address fe80::1 link-local
SLayer3(config-if)#description VLAN DATOS-2
SLayer3(config-if)#interface vlan 15
SLayer3(config-if)#ip address 192.168.25.1 255.255.255.224
SLayer3(config-if)#ipv6 address 2001:db8:acad:c::1/64
SLayer3(config-if)#ip6 address fe80::1 link-local
SLayer3(config-if)#description VLAN VoIP
SLayer3(config-if)#interface vlan 20
SLayer3(config-if)#ip address 10.100.1.1 255.255.255.248
SLayer3(config-if)#ipv6 address 2001:db8:cafe:a::1/64
SLayer3(config-if)#description VLAN ADMINISTRACION
SLayer3(config-if)#exit
SLayer3(config)#interface range g1/0/1-3
SLayer3(config-if-range)#switchport trunk encapsulation dot1q
SLayer3(config-if-range)#switchport mode trunk
SLayer3(config-if-range)#switchport trunk native vlan 25
SLayer3(config-if-range)#switchport nonegotiate
```

En este switch multicapa primero configuramos para que actúe como router, dado que también estaremos configurando ipv6 lo habilitamos por default para el dispositivo, como este switch será el router para los hosts en la red debemos configurar una dirección para las vlans, por último las interfaces que están conectadas a los demás switches se debe configurar prar que admita la encapsulación dot1q y demás configuraciones.

> En algunos switch multicapa a veces no reconoce algunos comandos, una razón puede ser porque esta en modo 'switchport', debemos quitar este modo para que admita los comandos 'no switchport'

La configuración en cada uno de los switch es:
```html
S1(config)#sdm prefer dual-ipv4-and-ipv6 default
S1(config)#vlan 5
S1(config-vlan)#name Data-1
S1(config-vlan)#vlan 10
S1(config-vlan)#name Data-2
S1(config-vlan)#vlan 15
S1(config-vlan)#name VoIP
S1(config-vlan)#vlan 20
S1(config-vlan)#name Admin
S1(config-vlan)#vlan 25
S1(config-vlan)#name Native
S1(config-vlan)#interface range f0/1-5
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 5
S1(config-if-range)#interface range f0/6-11
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 10
S1(config-if-range)#interface range f0/12-16
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport voice vlan 15
S1(config-if-range)#switchport access vlan 5
S1(config-if-range)#mls qos trust cos
S1(config-if-range)#interface f0/24
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 25
S1(config-if)#switchport nonegotiate
S1(config-if)#interface vlan20
S1(config-if)#ip add 10.100.1.2 255.255.255.248
S1(config-if)#ipv6 address 2001:db8:cafe:a::2/64
S1(config-if)#ipv6 address fe80::1 link-local
S1(config-if)#description VLAN ADMINISTRACION
S1(config-if)#exit
S1(config)#ip default-gateway 10.100.1.1
```
Habilitamos para que ipv6 sea admitido en el dispositivo, creamos vlans pero hay una vlan que es de VoIP por tanto esta vlan estará asignada en el rango de FastEthernet 0/12-16 también para que esta vlan puede tener acceso a datos y demás le damos acceso por la vlan 5 de Data-1 además que por esta vlan pasará voz se le da QoS (Quality of Service).

> Estas configuraciones deben ser iguales en el 'S1', 'S2' y 'S3' solo debemos poner la ip correspondiente.

#### Comprobaciones

En los hosts podemos comprobar la comunicación con 'pings' o incluso 'traceroute', recordemos antes asignarles una dirección IP a cada uno.

### Router on a stick
Este es otro método de routing entre vlans que consiste en usar un solo enlace físico por el cual pasan múltiples redes, este método es bueno cuando no disponen de suficientes recursos en donde se implementará, pero hay una gran desventaja al usar este método y es que hay un único punto de falla que es el router.

![rbv](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjPHERb0qFYSE7gr2p0oSYrpvEgIP6RvknArEmDnRh5RaEv8Od4qH7WFPbJINbeuBJU6EKa8WVm7h2sUFc-fI-GBGB14QE9IUprmmjLyrOU8xpQK3MqrG2qEI8tHFsTvfYOg2x1nnRcVpvgc_hzF7gB7NW_IdTui-LVQx7hom6EyRxoSVUVrcwn4hEzLw/s518/rbv.JPG)

```html
R1(config)#in g0/0/0.5
R1(config-subif)#encapsulation dot1Q 5
R1(config-subif)#ip add 192.168.0.1 255.255.255.224
R1(config-subif)#description VLAN-5
R1(config-subif)#interface g0/0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip add 172.16.0.1 255.255.0.0
R1(config-subif)#description VLAN-10
R1(config-subif)#interface g0/0/0.15
R1(config-subif)#encapsulation dot1Q 15
R1(config-subif)#ip add 192.168.5.1 255.255.255.0
R1(config-subif)#description VLAN-15-VoIP
R1(config-subif)#interface g0/0/0.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip add 10.10.14.1 255.255.255.248
R1(config-subif)#description VLAN-20-IT
R1(config-subif)#interface g0/0/0.25
R1(config-subif)#encapsulation dot1Q 25 native 
R1(config-subif)#description VLAN-25-Native
R1(config-subif)#in g0/0/0
R1(config-if)#no shutdown 
```
En el router este método de configuración va por subinterfaces y por ello no se debe encender cada subinterfaz porque son tomadas como 'interfaces virtuales', antes de poner la IP debemos poner el modo de encapsulación que debe hacer, cuando se termine de poner cada parámetro en las subinterfaces vamos a la interfaz física y la encendemos.

La configuración en el switch:
```html
S1(config)#vlan 5
S1(config-vlan)#name DATOS-1
S1(config-vlan)#vlan 10
S1(config-vlan)#name DATOS-2
S1(config-vlan)#vlan 15
S1(config-vlan)#name VoIP
S1(config-vlan)#vlan 20
S1(config-vlan)#name IT
S1(config-vlan)#vlan 25
S1(config-vlan)#name NATIVE
S1(config-vlan)#exit
S1(config)#in vlan 20
S1(config-if)#ip add 10.10.14.2 255.255.255.248
S1(config-if)#description VLAN-20-IT
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#interface f0/1
S1(config-if)#switchport mode trunk 
S1(config-if)#switchport trunk native vlan 25
S1(config-if)#switchport nonegotiate 
S1(config-if)#exit
S1(config)#ip default-gateway 10.10.14.1
S1(config)#interface range f0/2-6
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 10
S1(config-if-range)#exit
S1(config)#interface range f0/7-12
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport voice vlan 15 
S1(config-if-range)#switchport access vlan 5
S1(config-if-range)#mls qos trust cos
S1(config-if-range)#exit
S1(config)#interface range f0/13-22
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 5
```
En este punto los hosts de la red ya pueden comunicarse y la asignación de direcciones debe ser según el rango de las vlans asociadas a las interfaces.

#### Comprobaciones
```html
show interface trunk
show vlan brief
show dot11 associations
```
Con estos comandos podemos ver los que hemos configurado en el switch.
```html
show ip interface brief
```
Con este comando veremos el estado de las interfaces y las direcciones asignadas

Por último en los hosts podremos ver la configuración de red con 'ipconfig' podemos agregar '/all' para ver con detalle.

Cuando queremos seguridad en las vlans, podemos hacer una vlan para que las interfaces inactivas vayan asociadas a esta y apaguemos el rango de interfaces sin uso
```html
vlan 100
    name hold
interface range f0/19-24
    switchport mode access
    switchport access vlan 100
    shutdown
```

También para permitir que por el enlace truncal pasen algunas vlans que específiquemos es:
```html
interface range (rango o interfaz)
    switchport trunk allowed vlan 5, 12, 10 (vlans que necesiten pasar)
```

## [Atrás](./)