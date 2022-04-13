---
title: Routing
layout: default
---
*Static routing* es el proceso que tienen los routers de mover paquetes de un lado a otro, básicamente es el proceso de hacer comunicación de punto A a punto B y viceversa y esto es posible por medio de la red de internet. Existen diferentes formas para configurar enrutamiento como por protocolos de routing y enrutamiento estático y entre estas existen varios tipos de configuración.

* Rutas de siguiente salto
* Rutas flotantes
* Ruta de cuadrúple cero
* Ruta de host

Este tipo de rutas es común verlas configuradas en redes pequeñas debido a que no son muy grandes y no requiere de tanta configuración en cada router, este tipo de rutas no es escalable. Pero también las rutas estáticas tienen ciertas ventajas como que consumen menos ancho de banda y no utiliza CPU del dispositivo para calcular una ruta.

### Rutas de siguiente salto (next hop)
Se declaran rutas y el medio de paso es el router vecino, se puede poner la IP del router vecino o la interfaz
```html
R1(config)#ip route 172.16.0.0 255.255.0.0 10.10.10.2
R1(config)#do show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route
```
La salida del comando show ip route muestra las rutas configuradas en el router, en los dispositivos cisco podemos identificar que tipo de ruta es por la letra de código que pone al inicio, para este tema solo estaremos configurando rutas estáticas next hop (S).

Con IPv6 las rutas se configuran de forma similar:
```html
R1(config)#ipv6 route 2001:db8:acad:a::/64 2001:db8:cafe:c::b
R1(config)#do show ipv6 route
Codes: C - Connected, L - Local, S - Static, R - RIP, B - BGP
       U - Per-user Static route, M - MIPv6
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF intra, OI - OSPF inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
       D - EIGRP, EX - EIGRP external
```
También podemos poner rutas e idicar la interaz por la que tiene que pasar:
```html
R1(config)#ip route 10.12.13.0 255.255.240.0 serial 0/1/0
R1(config)#ipv6 route 2001:db8:acad:1::/127 serial 0/1/0
R1(config)#ipv6 unicast-routing
```
### Rutas flotantes
Este tipo de rutas son de respaldo y van por distancias administrativas, estas distancias van de 1 a 255 y su funcionamiento es mientras mayor sea la distancia la hace menos deseable, y mientras sea menor la hace mas deseable 
```html
R1(config)#ip route 10.10.14.0 255.255.240.0 10.15.14.3 5
R1(config)#ip route 172.100.1.0 255.255.254.0 serial 0/1/0 9
R1(config)#ipv6 route ::/0 2001:db8:cafe:1::c 4
R1(config)#ipv6 route 2001:db8:acad:a::/64 gigabitethernet 0/0/0 7
R1(config)#ipv6 unicast-routing
```
Aquí conseguimos que que si el equipo de un enlace deja de funcionar va a entrar en funcionamiento la otra ruta.

### Rutas de cuadrúple cero (defualt)
Son usadas cuando se descooce la ruta de destino de un paquete, este tipo de rutas da paso a todo tráfico por lo que presenta un riesgo de seguridad, podemos 'solventar' este riesgo con IPS, ACL o firewalls dedicados.
```html
R1(config)#ip route 0.0.0.0 0.0.0.0 10.10.1.2
R1(config)#ip route 0.0.0.0 0.0.0.0 serial 0/1/0
R1(config)#ipv6 route ::/0 2001:db8:cafe:c::a
R1(config)#ipv6 route ::/0 serial 0/1/0
R1(config)#ipv6 unicast-routing
```
> Al asignar estas rutas al router en la salida del comando show ip route y show ipv6 route debe mostrar que hay rutas así con un (*).

### Rutas de host
Son empleadas para ser mas eficaces con el proceso de paquetes debido a que son configuradas para un host específico, en un router se indentifican con (L)
```html
R1(config)#ip route 192.168.0.11 255.255.255.255 10.15.14.3
R1(config)#ip route 172.16.100.1 255.255.255.255 serial 0/1/0 5
R1(config)#ipv6 route 2001:db8:acad:2::15/128 serial 0/1/0
R1(config)#ipv6 route 2001:db8:acad:1::20/128 2001:db8:acad:c::c 4
R1(config)#ipv6 unicast-routing
```
> Este tipo de rutas se pueden combinar con otras, como las flotantes.

## [Atrás](./)