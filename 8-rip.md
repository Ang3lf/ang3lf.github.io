---
title: RIP
layout: default
---
*RIP* es un protocolo de gateway interno que intercambia información de rutas entre los dipositivos conectados. Este protocolo de routing es usado en redes pequeñas o medianas, algunas de sus caraceterísticas son:

* La distancia administrativa para la versión 2 es 120
* Multicast 224.0.0.9
* Actualmente la mayoría solo usa la versión 2
* Tiene un máximo de 15 saltos, en caso que la red tenga más las paquetes van de forma aleatoria provocando loops
* Actualizaciones limitadas, rip siempre envía toda su tabla de rutas en cada acualización
* Este protocolo en actualizaciones solo soporta 25 prefijos

Algunas deventajas son:

* La versión 2 va sumarizando las actualizaciones de red de forma automática
* Para decidir la mejor métrica toma en cuenta los saltos y descarta otros aspectos como bandwidth, latencia, fiabilidad, entre otros.

### Tabla de routing

* *Update time:* Cada router envía a su vecino todas las actualizaciones cada 30 segundos.

* *Invalid time:* En caso de que un enlace de un router cae o falla, el router vecino esperará 180 segundos para hacer la actualización de la nueva ruta a tomar.

* *Flush time:* Este se basa en 'invalid time' transcurrido ese tiempo, espera otros 240 segundos para poder eliminar esa ruta en particular de la tabla de enrutamiento, lo cual hace que todo el envío de paquetes sea lento ya que es mucho tiempo.

### Configuración

![rip1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjfXpSEQQLHT0dDjDUIS4tXUuWGMvrOOTZz8vP1tnr6UeetHgwPYkbLNSiICsBtEAX4j0cr84kipwRx3yWfJqM3fHz2MLQZHw5f6QWmHfIJGS6SXzpJzukMjXdnBekPGxnJJsGcIFexGsRYSbDj956SbEGErYmnKm-PtGC29zG0mcA6P_Gt0tc1uBVcsw/s1413/rip2.png)

```html
R1(config)#router rip
R1(config-router)#vesion 2
R1(config-router)#network 10.10.14.0
R1(config-router)#network 10.10.14.4
R1(config-router)#network 192.168.0.0
R1(config-router)#passive-interface default
R1(config-router)#no passive-interface s0/1/0
R1(config-router)#no passive-interface s0/1/1
R1(config-router)#no passive-interface s0/2/0
R1(config-router)#no auto-summary
```
En la imagen podemos ver las redes, la configuración no es nada compleja, solo debrmos declarar las redes sin la máscara, ponemos que las interfaces pasivas sean todas pero exceptuamos algunas que son las que están dentro de RIP, por último deshabilitamos la autosumarización para que no abarque todo el segmento de red y no descarte paquetes de esas redes.

De manera similar está configuración debe ir en los otros 3 routers, cuando terminemos de configurar en los routers deberíamos de tener comunicación con loopback del R1 a la red 192.168.0.0/24.

### Funcionamiento

![hops](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiZvaUfIaC6jYl8mdGJAPlpzhan2tP5OBhMjUzV6KJQYRCVDZw3AjbXSxFccwJ4_p8s2je4TWSGdZIRuM3xGIVt-mUFUEphcVo0jVmG_0I-fJ1apwtWweFhfDr1M51S-k_LhluxjjgGcb_Vzm7Pt7sW4840QpDyAjfzqD_nLdJx1XQ9qUNX6eouCWOLdQ/s950/rip1.png)

El funcionamiento de RIP es que busca la ruta que menor saltos tenga, por ejemplo si R1 quiere llegar a R4, va a disponer de 3 rutas alternativas, pero si pasa por ejemplo en R3 o R2 daría 2 saltos, mientras que si llega directo a R4 solo da un único salto.

![rip-hop](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiXZ7sZEmCel422o8VHEKnSd3EFhwxmlkW9Xi0CAHnd6eAsl4-CoeyHYxKwHDpsLQ5VSZ7yBJocLNy-R2r__bFtg79EI_xHbbUsk16Qwmn21xXT8FT9hnCh7ErMVXGT6JP8xmbMKHZDiwJyxRPVT9AvCha-76spiS01dJCytXooeh3L1_Tae5uoUL6g_A/s950/rip3.png)

Por tanto el paquete tomará la ruta conectada directamente.

Ahora supongamos que R1 nuevamente quiere llegar a R4 pero ahora solo dispone de 2 rutas

![rip-two](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiW9t6gBMXFQQ1r98igJqwNyAu2CZJerq875WU_jHV2_kLsanFTay_n-euwkN2MzNEQJRXakFMrYTzROTqChgohRl8fZOcnuddvD7Z5UqcMjLT4n85kjFWv-16cc8udhpSMucvaTlnSFFuZknqDSkZ4z_sxOIUu82S7pgTeJTyiKLhjzhGekVaw1O8yug/s950/rip4.png)

Como vemos en la imagen ahora ambas rutas tienen el mismo número de saltos, por tanto RIP hace que se envíen los paquetes de forma simultánea y así hace gestionar un equilibrio de carga y los datos llegan con menos lentitud.

### Adicional

Podemos configurar RIP para que ditribuya otros protocolos de routing
```html
R1(config-router)#redistribute ('ospf-eigrp-bgp-static-connected')
```

Cabe mencionar que si ponemos un protocolo como OSPFM EIGRP o BGP debemos poner seguido de este el AS que tiene el protocolo.

Podemos ver la rutas aprendidas con
```html
show ip rip database
show ip route -> buscar la salida que empiece con 'R'
```

Si queremos que RIP distribuya una ruta predeterminada fuera de las interfaces en las que está habilitada debemos poner la opción dentro del proceso RIP
```html
R1(config-router)#default-information
```
Esto hará que otros routers que tengan RIP puedan entran en el proceso RIP. 

## [Atrás](./)