---
title: OSPF
layout: default
---
*OSPF* es un protocolo de routing el cual busca llevar los paquetes por la ruta más corta, cada router configurado con este protocolo conoce la topología lógica de la red y es posible mediante mensajes que envían cada router configurado con el protocolo.

* LSU: (Link Status update) son mensajes que se envían entre los routers para mantenerse actualizados sobre posibles cambios que puedan ocurrir en la topología.
* LSA: (Link Status Advertaisement) son mensajes que llevan información sobre los router vecinos.
> Este tipo de datos van para llenar a una LSBD ( (Link State Database))

Este tipo de protocolos es común verlos funcionando en redes medianas incluso grandes junto con otros protocolos y es que este tipo de protocolos es escalable debido a que los cambios en la topología de red es dinámico. En OSPF las comunicaciones entre los routers point to point va por la IP 224.0.0.5 para los todos los routers agregados, y para los routers DR y BDR la dirección es 224.0.0.6.

### Estados del proceso OSPF
Son una serie de procesos que tiene OSPF para poder formar adyacencias con sus vecinos y hacer los procesos de elección.

1. *Down:* Primer estado del vecino ospf y es cuando no ha recibido paquetes de saludo de un vecino, pero hay ocaciones en que el vecino esta configurado manualmente para remover la configuración del estado del vecino es ahí donde este vecino se clasifica como full down.

2. *Init:* En este estado los routers reciben paquetes de saludo hello, estos paquetes incluyen el hello y ID del router, este id se agrega en el proceso ospf, si un router recibe un paquete hello pero no viene el ID, el router no puede pasar al siguiente estado.

3. *Bidireccional:* En este estado los routers ya han establecido comunicación porque cada router envió y recibió paquetes hello junto con sus IDs, también aquí es donde el router forma la adyacencia con el vecino, también en los medios broadcast un router se llena de todos los paquetes hello de toda la red que es el DR y el BDR es el router que mantiene esa comunicación bidireccional con el resto de los vecinos, y en una red punto a punto o multipunto a punto, cada router se llena de paquetes hello de sus vecinos, esto puede provocar cierto retraso en la red. Finalizando esta etapa en una red broadcast se elije un DR y BDR.

4. *Extart:* Aquí se empieza a establecer una relación 'maestro-esclavo' tomando en cuenta el 'router-id' más alto de todos los routers de la red, el router con ID más alto se vuelve *maestro* y este router es el único que puede icrementar su secuencia de paquetes, un aspecto a destacar es que si el DR se elije por el número de prioridad mas alto aun así este DR puede ser esclavo ya que en la elección de maestro se toma en cuenta el 'router-id' pero si no se configura una prioridad en los routers.

5. *Intercambio:* Los routers intercambian paquetes DBD (Descriptor Database) que contienen LSA y LSU con el fin de comprobar el estado de los enlaces.

6. *Loading:* En este estado se intercambia información real del estado de los enlaces basándose en los DBD que se repartieron del estado anterior, cada router ya conoce a su vecino con su ID.

7. *Full:* En este estado los routers ya son adyacentes, todos los LSA de los routers se intercambian en la red junto con las DBD para que todos estén sincronizados completamente.

Hay otro estado que se conoce como *Attempt* y es un estado que solo es válido para los vecinos configurados manualmente en un entorno NBMA (Network Broadcast Multiple Acsess) que es donde el router envía paquetes hello unicast en cada intervalo de sondeo del vecino y de quien no se recibe paquetes hello esta en un intervalo muerto.

Para que no haya retrasos o bucles en la red teniendo configurado OSPF debemos configurar algunos routers para que sean DR y BDR:
* *DR:* (Designated router) es el encargado de recibir y administrar los LSU y LSA que envía cada router conectado, este router se encarga además de garantizar que todos los routers conectados tengan LSBD idénticos.

* *BDR:* (Backup Desginated router) es el router de respaldo para el DR.

Se elije un DR y BDR conforme a la prioridad que tenga asignada en su interfaz, esta prioridad va de 0 a 255 y se toma el que tenga el valor más alto, hay situaciones en las que la prioridad no es configurada, entonces ospf se encarga obtener estos datos de otras maneras, si no hay prioridad asiganada busca 'router-id' que se configura en el proceso ospf, si no hay 'router-id' busca la dirección IP de la interfaz virtual (loopback), por última instancia toma la dirección IP de una interfaz física, este último no es bueno que quede configurado debido a posibles retrasos conflictos y atrasos con los paquetes.

```html
R1(config-router)#router-id 3.3.3.3
R1(config-if)#ip ospf priority 255
```
En una interfaz conectada con un vecino podemos configurar el tiempo y costo para trabajar con los vecinos, tiempo es el valor que podemos modificar para que se envíen los paquetes 'hello' entre cada vecino para comprobar su estado. Costo es la velocidad a la trabajan las interfaces entre los vecinos.
```html
R1(config-if)#ip ospf hello-interval 10
R1(config-if)#ip ospf dead-interval 40
```
Si pasa este tiempo y el vecino no reponde, los routers en la red deben actualizar LSBD hasta que el router se restablezca o se reemplace.
> Es importante saber que si modificamos los tiempos de los paquetes de saludo entre los vecinos, ambos routers deben tener el mismo tiempo configurado para que OSPF funcione, en la configuración mostrada esos son los tiempos predeterminados que trae OSPF.

_En un simulador como Cisco Packet Tracer podemos ver cómo se envían los mensajes de saludo en el modo de 'Simulación' filtrando por OSPF en la pestaña IPv4._

### Configurando OSPF

![ospf](https://www.firewall.cx/images/stories/ospf-operation-basic-advanced-concepts-ospf-areas-roles-theory-overview1.png)

Imaginemos que esta topologia la tenemos en nuestra empresa, y lo que nos piden que todas las redes de los usuarios tengan el mejor servicio posible al estar navegando en internet, entonces podríamos pensar en emplear ospf, y tomando en cuenta que hay un router intermediario para la conectarnos con el ISP, los únicos routers que estarían en particpando en OSPF sería R1, R2 y R3

```html
R1(config)#router ospf 5
R1(config-router)#router-id 1.1.1.1
R1(config-router)#network 10.0.0.0 0.0.0.3 area 0
R1(config-router)#network 10.0.0.8 0.0.0.3 area 0
R1(config-router)#passive-interface s1/0/1
R1(config-router)#auto-cost reference-bandwidth 1000
R1(config-router)#default-information originate
R1(config-router)#interface Serial0/1/0
R1(config-if)#ip ospf 5 area 0
R1(config-if)#ip ospf hello-interval 20
R1(config-if)#ip ospf dead-interval 80
```
En un proceso ospf podemos poner valores hasta 65535 pero se considera como buena práctica tener el mismo proceso en cada router que esté en el proceso OSPF, ya dentro del proceso 'ospf 5' indicamos el 'router-id', 'network' que es donde anunciamos las redes vecinas junto con el wildcard correspondiente,que wildcard en pocas palabras sirve para indicar el tamaño de una red o una subred, 'passive-interface' indica que la interfaz s1/0/1 no recibirá ningun paquete sobre OSPF, 'auto-cost reference-bandwidth 1000' indicamos que el costo de la interfaz GigabitEthernet sea 10Gbps y para FastEthernet sea 100 Mbps y 'default-information originate' hace distribuye automáticamente la ruta predeterminada a todos los routers de la red. Ya para finalizar en las interfaces que estén conectadas a otros routers con OSPF asignamos el proceso ospf correcto junto con el área.

##### Calcular wildcard
Una wildcard es fácil de calcular, tenemos que restar /32 - la mácara que tengamos en nuestra red, ejemplo:

| 255.255.255.255|
|:---------------|
|-255.255.255.248|
|  0 . 0 . 0 . 7 |

Área puede ser configurado según lo soliciten, pueden existir varias áreas, hay que tener en cuenta al configurar varias áreas debemos tomar en cuenta en que router se configura, por ejemplo donde finalice el área 0 habrá un router entonces ahí dependiendo de las interfaces conectadas se pone el proceso OSPF y el área según corresponda, normalmente este tipo de router se les conoce como ABR (Area Border Router).

## [Atrás](./)
