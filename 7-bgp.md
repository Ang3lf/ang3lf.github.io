---
title: BGP
layout: default
---
*BGP* es un protocolo de routing externo, este es diferente a los otros como OSPF, EIGRP y RIP que trabajan de forma interna, además que BGP usa un tipo de mensajes 'Path vector' e incliye una ruta completa a cada destino que va dando la información mediante sistemas autónomos.

Este protocolo usa el puerto 179/TCP, cada dispositivo se logra identificar por el router-id al igual que OSPF. Otra característica de BGP es que usa toda la información sobre el routing en la red para poder mantener una base de datos y ver el alcance que tiene sobre la red, también usa esta información para poder construir su tabla de topología y poder eliminar bucles de routing y tomar la decisionesde la directiva siempre en el nivel de AS (Autonumous System).

#### Modos BGP

BGP trabaja de distintas formas, se puede usar IBGP (Internal Border Gateway Protocol) y EBGP (External Border Gateway Protocol), estos modos se pueden implementar en varios escenarios, por ejemplo IBGP si que queramos una red interna para que sea de la empresa y EBGP para que sea la que se comunica con el ISP sería un enlace punto a punto, claro en este escenario el administrador de la red debe pedir información al ISP sobre la IP que se trabajará, AS, etc.

![bgp](https://www.networkstraining.com/wp-content/uploads/2017/03/ebgp-ibgp.jpg)

#### Mensajes BPG

* *Open message:* Luego de que establecen la conexión en TCP, ya pueden empezar a crear una conexión BGP entre ellos e intercambian tráfico de datos.

* *Update message:*  Transfieren información de routing entre pares sobre BGP, usan esta información para crear un gráfico que muestre las relaciones entre los routers vecinos.

* *KeepAlive message:* Son los que mantienen las conexiones BGP, verifican si un enlace de algún vecino ha fallado o ya no se encuentra disponible, estos mesnajes se intercambian cada cierto tiempo.

* *Notification message:* Mensajes cuado se detectan errores en el proceso BGP, esto hace que en algunos casos cierre la sesión BPG.

* *Route-refresh message:* Solicitan de forma automática a un vecino de ruta BGP que reenvíe mensajes de actualización. 

#### Configuración básica BGP

![topobpg](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgXDeFRY-8UvXdII_1NeEOn5e5lNfm5EgQSYGXPBA9M8NsLdKWzQMukt1xFO9isbl5MtxNP6HARvQKpTS88-mp1QvlE4Mt8vJ4kZa_bFdBOiIh1DCXzjkcSescKRbWWHQPUEDb9wsezSeqQ5Pg_DSNj7QaTvJpWFS0iMPTGKMvO5NiEB0z7uNIy06UZyQ/s775/bgp.JPG)

Estaremos tomando en cuenta esta topología para ir haciendo la configuración, podemos configurar los 2 modos EBGP e IBGP. EBGP se configurará en el enlace entre R1 y el ISP, IBGP se configurará en los routers R1-R2-R3 (En el simulador packet tracer no es posible trabajar con IBGP solo con EBGP).

##### EBGP -> R1-ISP
```html
R1(config)#ipv6 unicast-routing
R1(config)#router bgp 115
R1(config-router)#bgp router-id 1.1.1.1
R1(config-router)#no bgp default ipv4-unicast
R1(config-router)#neighbor 209.165.100.1 remote-as 1000
R11(config-router)#network 209.165.100.0 mask 255.255.255.248
R1(config-router)#address-family ipv4 unicast
R1(config-router-af)#neighbor 209.165.100.1 activate
R1(config-router-af)#exit
R1(config-router)#neighbor 2001:db8:cafe:a::1 remote-as 1000
R1(config-router)#network 2001:db8:cafe:a::/125
R1(config-router)#bgp log-neighbor-changes
R1(config-router)#no synchronization
```

Lo primero en configurar es el routing ipv6, esto se debe poner de primero para que al momento de declarar una ruta ipv6 no muestre un error, ponemos el número de proceso bgp (AS), después un 'router-id' que identifica al dispositivo. Con 'no bgp default ipv4-unicast' indicamos que trabaje solo con ipv4, queremos que trabaje ipv6 también. En 'neighbor' indicamos la ip del vecino bgp junto con el AS que tenga. 'network' declaramos la red junto con la máscara. 'address-family ipv4 unicast' aquí especificamos la familia de direcciones ipv4 de vecinos, al estar dentro de la familia de direcciones ponemos la ip del vecino declarando para que lo tenga activo. Luego se declaran las redes ipv6 junto con AS y máscara. Por último con 'bgp log-neighbor-changes' indicamos que queremos habilitar la generación de mensajes de registro de cambios de vecinos BGP como por ejemplo; si se restablece, se activa o desactiva. 'no synchronization' indica que no queremos que anuncie mensajes sobre lo que aprenda de un vecino IBGP a otro vecino EBGP en caso deno ser validado.

```html
ISP-1(config)#router bgp 1000
ISP-1(config-router)#bgp router-id 2.2.2.2
ISP-1(config-router)#neighbor 209.165.100.3 remote-as 115
ISP-1(config-router)#no bgp default ipv4-unicast
ISP-1(config-router)#network 209.165.100.0
ISP-1(config-router)#address-family ipv4 unicast
ISP-1(config-router-af)#neighbor 209.165.100.3 activate
ISP-1(config-router-af)#exit
ISP-1(config-router)#neighbor 2001:db8:cafe:a::3 remote-as 115
ISP-1(config-router)#network 2001:db8:cafe:a::/125
ISP-1(config-router)#bgp log-neighbor-changes
ISP-1(config-router)#no synchronization
```
En el router ISP son similares las configuraciones, algo a destacar es que también podemos declarar las redes solo poniendo el prefijo sin la máscara.

##### IBGP R1-R2-R3
```html
R1(config)#router bgp 115
R1(config-router)#neighbor 10.10.14.6 remote-as 115	
R1(config-router)#network 10.10.14.4 mask 255.255.255.252
R1(config-router)#no synchronization 
R1(config-router)#bgp log-neighbor-changes 
R1(config-router)#network 10.10.14.8 mask 255.255.255.252
R1(config-router)#neighbor 10.10.14.10 remote-as 115
R1(config-router)#network 2001:db8:acad:c::/127
R1(config-router)#neighbor 2001:db8:acad:c::2 remote-as 115
R1(config-router)#network 2001:db8:acad:b::/127
R1(config-router)#neighbor 2001:db8:acad:b::2 remote-as 115
R1(config-router)#address-family ipv4 unicast
R1(config-router-af)#neighbor 10.10.14.6 activate
R1(config-router-af)#neighbor 10.10.14.10 activate
```

```html
R2(config)#router bgp 115
R2(config-router)#bgp router-id 3.3.3.3
R2(config-router)#no bgp default ipv4-unicast
R2(config-router)#neighbor 10.10.14.5 remote-as 115
R2(config-router)#network 10.10.14.4 mask 255.255.255.252
R2(config-router)#neighbor 10.10.14.2 remote-as 115
R2(config-router)#network 10.10.14.0 mask 255.255.255.252
R2(config-router)#address-family ipv4 unicast
R2(config-router-af)#neighbor 10.10.14.5 activate
R2(config-router-af)#neighbor 10.10.14.2 activate
R2(config-router)#no synchronization 
R2(config-router)#bgp log-neighbor-changes 
R2(config-router)#network 2001:db8:acad:c::/127
R2(config-router)#neighbor 2001:db8:acad:c::1 remote-as 115
R2(config-router)#network 2001:db8:acad:a::/127
R2(config-router)#neighbor 2001:db8:acad:a::2 remote-as 115
```

```html
R3(config)#router bgp 115
R3(config-router)#bgp router-id 4.4.4.4
R3(config-router)#no bgp default ipv4-unicast
R3(config-router)#neighbor 10.10.14.9 remote-as 115
R3(config-router)#network 10.10.14.8 mask 255.255.255.252
R3(config-router)#neighbor 10.10.14.1 remote-as 115
R3(config-router)#network 10.10.14.0 mask 255.255.255.252
R3(config-router)#address-family ipv4 unicast
R3(config-router-af)#neighbor 10.10.14.9 activate
R3(config-router-af)#neighbor 10.10.14.1 activate
R3(config-router)#no synchronization 
R3(config-router)#bgp log-neighbor-changes 
R3(config-router)#network 2001:db8:acad:a::/127
R3(config-router)#neighbor 2001:db8:acad:a::1 remote-as 115
R3(config-router)#network 2001:db8:acad:b::/127
R3(config-router)#neighbor 2001:db8:acad:b::1 remote-as 115
```

Los 'router-id' de cada router debe ser diferente para no crear conflictos.

##### Adicional
Podemos ver las configuraciones aplicadas con los siguientes comandos:
```html
show ip bgp
show running-config | section router bgp
```
El resultado del primero mostrará las rutas que se han formado por BGP
```html
R1#show ip bgp
BGP table version is 5, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.10.14.4/30     0.0.0.0                  0     0 32768 i
*> 10.10.14.8/30     0.0.0.0                  0     0 32768 i
*> 209.165.100.0/29  0.0.0.0                  0     0 32768 i
```
> En caso de que tengamos subredes y no queremos que se agregue todo el segmento podemos poner en el AS de bgp 'no auto-summary'.

## [Atrás](./)