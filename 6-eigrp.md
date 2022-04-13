---
title: EIGRP
layout: default
---
*EIGRP* es un protocolo de routing que ofrece los mejores algoritmos en las distancias, este protocolo es una versión mejorada de *IGRP*, ahora EIGRP tiene mejores tiempos de convergencia y eficiencia de paquetes, básicamente el algoritmo que usa se conoce como DUAL (Diffusing Update Algorithm) que es el que obtiene en cada instante el cálculo de una ruta haciendo que los routers en la red se sincronicen al mismo tiempo en caso que haya un cambio de rutas, también este algoritmo usa la tabla de vecinos y la tabla de topología para desarrollar la tabla de enrutamiento en el router EIGRP, algunas de las características de este protocolo son:
* Propietario: Cisco Systems
* Algoritmo: DUAL
* Tipo: Vectores de distancia
* Protocolo: RTP (Reliable Transport Protocol)
* IP Multicast address: 224.0.0.10
* MAC Multicast address: empiezan en 01-00-5E

En el modo de operación se usa AS (Autonomous System) para activarlo, AS; son un grupo de direcciones IP que pertenecen a una entidad específica como un ISP, gobiernos o instituciones privadas. También usa mensajes 'Hello' para mantener comunicación con los vecinos además que las actualizaciones son limitadas.

EIGRP tiene para un máximo de 255 saltos, por default en equipos Cisco viene configurado con 100 saltos, soporta sumarización manual en interfaces dentro del conjunto EIGRP es decir que se pueden por decirlo así 'repetir' las redes en el conjunto y que no haya necesidad de declarar red a red, cada router tiene 2 tablas una de topología y la otra de vecinos y con estas tablas deciden cuál es la mejor ruta a tomar para el paquete.

#### Mensajes EIGRP (Proceso)
1. Hello -> 5 segundos || Hold-interval -> 15 segundos
2. Update
3. ACK
4. Query
5. Reply

### Configuración

![eigrp](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhZS31qVNVPj_tXVgv6RAgaahWmbhES0vhvAcRrQzjIKaAe6U4P1C0isx5cmzxIEcmnM1VvfGiui3NPsmavXWC1ulUD4AEmQcuweMxHRaGiWTnGI4VMf9sf-ok_LlbkCiWn_2J66aiwcRSrkOyEffHH_wHFyWNb5-IY3c2B63SvGyxzDsEHRs8lfTTQfQ/s832/eigrp.JPG)

Tomando en cuenta la topología de la imagen vamos a configurar EIGRP en los routers, este protocolo se configura de manera similar que otros protocolos, primero le damos un número al proceso eigrp, declaramos las redes y apartamos interfaces para que queden fuera del proceso eigrp.
#### Router-1
```html
R1(config)#router eigrp 5
R1(config-router)#no auto-summary 
R1(config-router)#network 10.1.1.0 0.0.0.3 
R1(config-router)#network 10.2.2.1 255.255.255.252 
R1(config-router)#passive-interface default 
R1(config-router)#no passive-interface g0/0/0
R1(config-router)#no passive-interface g0/0/1
```
Primero en la línea 2 el comando 'no auto-summary' indica que no quiere que se sumaricen las redes con el proceso eigrp, si no declaramos esto es muy probable que haya retrasos y conflictos en la red porque al dejar que se sumaricen las redes hace que por ejemplo la red de Lo4 172.16.10.0/28 sea parte de todo el conjunto de direcciones clase B (172.16.0.0/16). En 'network' declaramos las redes junto con su wildcard, también podemos declarar la red específica de un router en la interfaz que sale junto con la máscara. En las interfaces pasivas podemos declarar a todas las interfaces del dispositivo con 'default', pero siempre hay interfaces las cuales queremos que si reciban paquetes eigrp y en este caso son las interfaces g0/0/0-1.

#### Router-2
```html
R2(config)#router eigrp 5
R2(config-router)#no auto-summary
R2(config-router)#network 10.1.1.0 0.0.0.3
R2(config-router)#network 10.0.0.2 0.0.0.0
```
El proceso eigrp debe ser el mismo que el de los otros routers de la red, y otra vez 'no auto-summary' para que el dispositivo no piense que ya la conoce y descarte el paquete. En la última declaración de 'network' es otra forma de especificar la dirección IP de donde sale el proceso eigrp pero ahora la wildcard decimos que es solo de esa interfaz con la IP que pusimos. 

#### Router-3
```html
R3(config)#router eigrp 5
R3(config-router)#no auto-summary 
R3(config-router)#network 0.0.0.0
R3(config-router)#passive-interface loopback 4
R3(config)#ip route 0.0.0.0 0.0.0.0 209.165.100.1
R3(config)#do ping 8.8.8.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
```
Las configuraciones eigrp son parecidas la de los otros routers, en 'network 0.0.0.0' esa configuración hace que habilite eigrp en todas las interfaces que tengan una IP configurada y estén estado'up', claro está configuración es recomendable hacerla en un entorno controlado donde sabemos que tipo de tráfico fluye en la red y que tipo de personas se conectan. La ruta estática declaramos en el router que está conectado con el ISP, debemos configurar una ruta default para permitir el acceso a internet, luego de poner la ruta podemos comprobar la comunicación con un ping a la Lo3 del ISP 8.8.8.8.

#### Router-4
```html
R4(config)#router eigrp 5
R4(config-router)#passive-interface lo5
R4(config-router)#no auto-summary 
R4(config-router)#network 0.0.0.0
```
Mientras estemos configurando eigrp en los router nos mostrará mensajes por consola sobre las adyacencias que ha ido formando, en router-4 ya llegamos a la Lo3 del ISP.

### Otras configuraciones

Hasta el momento ni uno de los routers pueden llegar a el ISP a excepción claro de R3 y también algunos routers de la red no logran llegar a las loopbacks, este es la tabla de rutas del router-4:
```html
R4#show ip route eigrp
     10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
D       10.0.0.0/30 [90/2170112] via 10.10.14.1, 01:05:53, GigabitEthernet0/0/0
D       10.1.1.0/30 [90/3072] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
D       10.2.2.0/30 [90/3328] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
     172.16.0.0/28 is subnetted, 1 subnets
D       172.16.10.0 [90/131328] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
D    192.168.0.0/24 [90/130816] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
     192.168.1.0/26 is subnetted, 2 subnets
D       192.168.1.0 [90/131072] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
D       192.168.1.64 [90/131072] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
     209.165.100.0/29 is subnetted, 1 subnets
D       209.165.100.0 [90/3584] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
D*EX 0.0.0.0/0 [170/5888] via 10.10.14.1, 01:06:01, GigabitEthernet0/0/0
```
También podemos ver lo que hemos configurado de eigrp en los router poniendo lo siguiente:
```html
show running-config | section eigrp
```

Modificar tiempos de saludo y espera:
```html
ip hello-interval eigrp 5 15
ip hold-time eigrp 5 45
```

Podemos hacer que los router tengan comunicación con cada una de sus interfaces poniendo una redistribución estática en el router borde, para que los routers usen como 'gateway' a R3 y así puedan llegar al ISP.
```html
R3(config)#router eigrp 5
R3(config-router)#redistribute static
R3(config-router)#in g0/0/1
R3(config-if)#ip summary-address eigrp 5 192.168.1.0 255.255.255.192
R3(config-if)#in s0/2/0
R3(config-if)#ip summary-address eigrp 5 192.168.1.0 255.255.255.192
```
En la configuración de la interfaz estamos agregando una sumatoria de direcciones en el proceso eigrp, esta sumatoria la podemos calcular reduciendo la tabla de enrutamiento agarrando al red inferior y colocándolas en las 2 interfaces del router borde.

> En caso de que tengamos configuradas más rutas default, podemos aplicar 'road-map' para ir filtrando.

##### ¿Qué pasaría si el enlace entre R2 y R3 cae?

En estos casos es donde EIGRP deja de recibir paquetes 'hello' en ese enlace, pasa el tiempo configurado de espera para recibir el paquete y mandan paquetes update al router vecino, los routers afectados empiezan a realizar un recálculo de ruta y la nueva ruta calculda sería por el enlace de R2 y R1.

Algunos problemas pueden venir por inconsistencia en los routers, pueda ser que los tiempos de 'hello' y 'hold' no coincidan entre los routers. Podemos configurar manualmente los saltos:
```html
router eigrp 5
     metric maximun-hops 1-255
```
Podemos ajustar los valores de ancho de banda para el protocolo eigrp, es decir con esta configuración indicamos cuánto es lo máximo que puede usar eigrp en la red, va por porcentajes:
```html
R3(config-router)#in g0/0/1
R3(config-if)#ip bandwidth-percent eigrp 5 5
```
Con esta configuración hacemos que eigrp use para los paquetes 'hello', 'update', 'query', 'reply', 'ACK'. Se considera buena práctica dejar un 5% de bandwidth para este protocolo. También podemos configurar 'metric weight' que son la métricas estáticas que se configuran en los routers para que puedan trabajar en la red. Estos valores se identifican con una K:
* *K1:* Es el valor del ancho de banda
* *K2:* Es la carga de paquetes
* *K3:* Va relacionado a los retrasos
* *K4:* Es la confiabilidad sobre las interfaces
* *K5:* Es MTU (Maximun Transfer Unit) que expresa el tamaño en bytes de los datos más grandes que puede enviarse usando un protocolo de comunicaciones
```html
R3(config)#router eigrp 5
R3(config-router)#metric weights 0 1 1 1 0 0
R2(config-router)#metric weights 0 1 1 1 0 0
R1(config-router)#metric weights 0 1 1 1 0 0
```
> Es importante saber que estos valores deben coincidir con los demás routers, para que no haya problemas al formar las adyacencias.

## [Atrás](./)
