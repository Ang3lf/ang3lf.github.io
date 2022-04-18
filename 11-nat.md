---
title: NAT
layout: default
---
*NAT* es el sistema usado en redes para traducir una dirección ip pública a una privada y viceversa con el objetivo de que los hosts de una red puedan salir de su LAN y comunicarse con otras, y es que NAT llegó para evitar que haya mismas direcciones IP andando en internet, NAT para nada es una forma de seguridad en red que oculta la ip privada, si alguien logra tener nuestra IP publica puede geolocalizar de donde viene con herremientas de algún repositorio o incluso online, logrando hacer actos maliciosos.

Debido al agotammiento de direcciones IPv4 se crea NAT para hacer que direcciones no enrutables en internet puedan salir de su red local y puedan interacturar con otras, con IPv6 ya no es necesario NAT ya que ipv6 tiene miles de millones de direcciones para cada host que se conecte a una red.

| Clase | Rango de direcciones privadas | Máscara de red |
|:------|:------------------------------|:---------------|
|   A   |  10.0.0.0 - 10.255.255.255    |   255.0.0.0    |
|   B   |  172.16.0.0 - 172.16.31.255   |  255.255.0.0   |
|   C   | 192.168.0.0 - 192.168.255.255 | 255.255.255.0  |

### Terminología

* *Direacción local interna:* Es la dirección de origen vista desde el interior de la red. 

* *Dirrección global interna:* Es la dirección que un router convierte para que pueda salir de la red e interactuar con otras, y es vista desde el exterior.

* *Dirección local externa:* Es una dirección pública vista desde su red interna.

* *Dirección global externa:* La dirección pública vista desde su red externa, podria ser 209.165.100.1 que puede ser enrutable en su red.

### Tipos de NAT

* *NAT estático:* Es usado para traducir direcciones de equipos, regularmente se usa para servidores en datacenters.

* *NAT dinámico:* Se usa para dar a un cojunto de host en la red una ip según vayan solicitando, se hace un 'pool' de direcciones.

* *PAT:* Este tipo es similar a NAT dinámico con una diferencia que es que usa la misma dirección ip para todos los host de la red pero usa un puerto diferente por cada host.

### Configuración

![p1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiD71sCPLSzDhY4yl9hSrKDgjogEqwsdiczkqMIwgUarNUBZknVwAT1QbbKSU0Ark2xAL0MkKyyMqqkpMuOCsVyRksxtoqn1M7M7bgAUeXGRxbv_inyMsQT2VTTzDwT8GReRLQ3mXodUl5BDBkDyHMg9Db-GBQUASFkCt8WOnuMv5gjXl-wySmIb3rNSw/s1466/p1.png)

La configuración es estática y para un servidor que regularmente es donde se configura este tipo de servicios, las direcciones que tiene cada interfaz son:

```html
R1(config)#do show ip interface g0/0/0
GigabitEthernet0/0/0 is up, line protocol is up (connected)
  Internet address is 10.0.0.1/27
R1#(config)do show ip interface s0/1/0
Serial0/1/0 is up, line protocol is up (connected)
  Internet address is 190.214.15.67/29
```

```html
R1(config)#ip nat inside source static 10.0.0.3 190.214.15.65
R1(config)#ip access-list standard NAT
R1(config-std-nacl)#permit 10.0.0.0 0.0.0.31
R1(config-std-nacl)#deny any
R1(config-std-nacl)#exit
R1(config)#interface g0/0/0
R1(config-if)#ip nat inside 
R1(config-if)#interface s0/1/0
R1(config-if)#ip nat outside 
```
Necesitamos crear una ACL para que se tenga más control sobre el tráfico. Luego en la interfaz debemos poner hacia donde traduce la dirección según la conexión con el vecino, en este ejemplo en la g0/0/0 está conectado a la LAN y en la s0/1/0 está conectada al ISP.

#### NAT dinámico

![p2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj33TxzyWgz-qx0OYp1gZMk7gyfBqlkKWNljtYvfQtfR2hpqwKW_N0twrawtqyhRCahK15amjuNDupTeRLHgadZfJSkXfQ8Iijd6OoWDadJ6SIoLnfqT0Pc8-69FvMT8iNA88IyDODQHGXd8GXOy0a34UuJtrtajdf2x5RvnujaNimOKrQgnjbVbC3voQ/s1466/p2.png)

Para configurar NAT dinámico necesitamos crear un pool de direcciones que estén disponibles para poder usar
```html
R1(config)#ip access-list standard NAT
R1(config-std-nacl)#permit 10.0.0.0 0.0.0.31
R1(config-std-nacl)#deny any
R1(config)#ip nat pool NAT 190.214.15.66 190.214.15.69 netmask 255.255.255.248
R1(config)#ip nat inside source list NAT interface s0/1/0
R1(config)#ip nat inside source list NAT pool NAT
R1(config)#interface g0/0/0
R1(config-if)#ip nat inside 
R1(config-if)#interface s0/1/0
R1(config-if)#ip nat outside 
```
Podemos ver NAT configurado
```html
R1#show ip nat translations 
Pro  Inside global  Inside local    Outside local    Outside global
icmp 190.214.15.65    10.0.0.3       190.214.15.70    190.215.15.70
icmp 190.214.15.66    10.0.0.11      190.214.15.70    190.214.15.70
```

#### PAT

![p3](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgGSibVpRMzLcgwFlvedcsc-I_1HbpM8qGgCwdTtrDiyokvsq0jGPC_u43CdEp0g0KUw-EYJuOjSiWUQUJJX7v-EDkmbfFjaQ5zFxHqwvNnoB9_xpuILLt7yanaSiqhORC5SC50Y8RD4Wc0qLI0hqpgJI1ho_Jz1sFQ7oycs3go5v2R--d5HD6-CvfDQA/s1466/p3.png)

Es la forma de configurar NAT reutilizando las direcciones ip de un pool, y esto es posible porque PAT usa diferente puerto para cada host en la red, la configuración es similar a configurar NAT dinámico

```html
R1(config)#access-list 1 permit 10.0.0.0 0.0.0.31
R1(config)#ip nat pool NAT 190.214.15.66 19.0.214.15.69 netmask 255.255.255.248
R1(config)#ip nat inside source list 1 pool NAT overload
R1(config)#ip nat inside source list 1 interface s0/1/0 overload
R1(config)#do show ip nat translations 
Pro  Inside global     Inside local       Outside local      Outside global
icmp 190.214.15.66:1024  10.0.0.2:15      190.214.15.70:15    190.214.15.70:1024
icmp 190.214.15.66:13    10.0.0.3:13      190.214.15.7013     190.214.15.70:13
icmp 190.214.15.66:14    10.0.0.11:14     190.214.15.70:14    190.214.15.70:14
icmp 190.214.15.66:15    10.0.0.12:15     190.214.15.70:15    190.214.15.70:15
```
Al final *overload* (sobrecarga) hace que se use la misma dirección IP y que la comumicación en redes externas sea por puertos diferentes, en la salida del comando 'show ip nat translations' podemos ver las traducciones que ha hecho, y comprobamos que la IP no cambia, lo único que cambia es el puerto para cada host.

Este método de configuración es común verlo en routers de casa donde es más conveniente usar solo una ip pública. 

## [Atrás](./)