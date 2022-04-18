---
title: ACL
layout: default
---
*ACL* son listas de control de acceso que determinan permisos de acceso a una red, estas se encargan de dar seguridad a l red debido al filtro de tráfico que realizan, este tipo de configuración es el encargado de restringir o dar acceso a cierto tráfico ya sea entrante o saliente, es un elemento programable.

Generalmente se recomienda hacer implementaciones de ACL en un entorno controlado o simulado de ser posible, ya que una mala configuración puede dejar sin acceso a toda la red, por ello debemos hacer varias pruebas antes de hacer la implementación real para obtener el resultado deseado.

### Tipos de ACL

* *ACL standard:*  Permiten o niegan tráfico basándose en la dirección de origen.
* *ACL extended:* Permiten o niegan paquetes según la IP, protocolo o puertos de origen o destino.

Podemos configurar de dos formas en ambos tipos de ACL, configuración de ACL nombrada o numerada. ACL numeradas:

* *Standard* puede ir de 1 a 99 y de 1300 a 1999. 
* *Extended* puede ir de 100 a 199 y de 2000 2699.

Con una ACL nombrada en ambos tipos podemos poner un nombre específico que la identifique con algunos aspectos a tomar en cuenta:

* El nombre de la ACL se recomienda que se ponga en mayúsculas
* No deben llevar espacios de por medio
* Podemos poner carácteres alfanúmericos

### Ubicación de ACL

* Una lista de acceso estándar se coloca lo más cerca del destino posible.
* Una lista de acceso extendida se coloca lo más cerca posible del origen.

Cuando estemos en la configuración de una interfaz del dispositivo debemos indicar hacia donde va la inspección del tráfico

* *IN* es el tráfico que llega a la interfaz y luego pasa por el router
* *OUT* el tráfico que ya pasó por el router y está por salir de la interfaz

Hay algunos acpectos a tomar en cuenta mientras se configuran ACL
1. No se puede tener 2 listas de acceso entrante de una interfaz
2. Se puede tener diferentes listas de acceso a una interfaz, una de entrada y otra de salida
3. Se debe seguir cierto orden para permitir o denegar, por ejemplo si queremos denegar a un solo host de toda una red primero debemos poner el host y luego le damos permiso a la red para que no traiga algo con anterioridad y sería lo mismo de la forma contraria

### Configuración

![a1](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjbyv3ACd5IOKcsdM0sFbKxSYEQ3zYNh9EwUXvxdlsatn30VraE_3GOdY86cPQBL5tGfLGXNIYhZQSYfFueTB0YD9OyeJ0zVkrgslB-8b138QorkLPLVSN_Utbn87HB2Y91Uukyz7Q0GXJsCLLK2BF2H8tHjo8-4rZVMIuXRGUnl3k0YS8D6ZUy4RFmVA/s1693/a1.png)

En este caso queremos impedir que el host de la red 192.168.5.0/28 tenga comunicación con la red 172.255.0.0/29, se ha configurado previamente routing estático en las redes, estas son las direcciones que tiene cada dispositivo de red en sus interfaces:
```html
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   192.168.5.1     YES manual up                    up 
GigabitEthernet0/0/1   192.168.0.1    YES manual up                    up 
GigabitEthernet0/0/2   10.0.0.1        YES manual up                    up 
```
```html
S1#show ip interface vlan 1
Vlan1 is up, line protocol is up
  Internet address is 192.168.5.2/28
```
```html
S2#show ip interface vlan1
Vlan1 is up, line protocol is up
  Internet address is 192.168.0.2/26
```
```html
R2#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   172.255.0.1      YES manual up                    up 
GigabitEthernet0/0/1   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/2   10.0.0.2        YES manual up                    up 
```
```html
S3#show ip interface vlan1
Vlan1 is up, line protocol is up
  Internet address is 172.255.0.2/29
```

#### ACL standard numbered
```html
R2(config)#access-list 1 deny host 192.168.30.5
R2(config)#access-list 1 permit any 
R2(config)#access-list 1 remark DENIED-HOST
R2(config)#interface g0/0/0
R2(config-if)#ip access-group 1 out
R2(config-if)#exit
R2(config)#do show access-lists 
Standard IP access list 1
    10 deny host 192.168.5.10 (10 match(es))
    20 permit any (2 match(es))
```
La ACL está configurada en R2 porque si recordamos sobre las 'ubicaciones de ACL' dice que una ACL standard se debe colocar lo más cerca del destino, y en este caso R2 es el más cercano a la red 172.255.0.0/29. *remark* indicamos que lo que sigue es un comentario para la ACL, *any* debemos tener cuidado cómo lo ponemos, si ponemos 'permit any' le indicamos que todo tráfico está permitido, si ponemos 'deny any' para negar todo el tráfico, por úlimo toca asignar hacia donde evalue el tráfico para la interfaz con ACL, en este caso se pone 'out' para que no inspeccione el tráfico saliente de la interfaz.

Con el comando 'show access-lists' podremos ver la actividad qye ha tenido la ACL, en este caso muestra en la salida que el host 192.168.5.10 ha intentado 10 veces comunicarse con un host de la red 172.255.0.0/29, y en la siguiente línea muestra que los hosts de otras redes se han comunicado con la red.

#### ACL standard named
```html
R2(config)#ip access-list standard DENIED-HOST
R2(config-std-nacl)#permit host 192.168.0.5 
R2(config-std-nacl)#deny host 192.168.30.5
R2(config-std-nacl)#exit
R2(config)#interface g0/0/0
R2(config-if)#ip access-group DENIED-HOST out
R2(config-if)#exit
R2(config)#ip access-list standard DENIED-HOST
R2(config-std-nacl)#30 permit any
R2(config-std-nacl)#end
R2#show access-lists 
Standard IP access list DENIED-HOST
    10 permit host 192.168.0.5 (3 match(es))
    20 deny host 192.168.30.5 (5 match(es))
    30 permit any (3 match(es))
```
La confguración de este tipo de ACL cambia un poco a comparación de la numerada, ahora debemos poner un nombre que sea único para la ACL, y entramos a la configuración de ACL donde colocamos todo lo que queremos hacer, se asigna a la interfaz que corresponda, puede darse el caso en que olvidemos permitir el demás tráfico, podemos editar una ACL en lugar de tener que crear otra, para editar la ACL solo debemos ingresar a la ACL y poner un número que esté dentro del rango, este rango lo podemos ver en la salida del comando 'show access-list', es importante escoger un número que no esté ocupado por la ACL es decir si 10 y 20 ya tienen un valor no debemos agarrar esos 2 números, debemos poner otro nuevo. Ya por último para asignar este tipo de ACL a la interfaz es similar a una numerada lo que cambia es el nombre de la misma.

> Para permitir o denegar toda la red debemos poner la dirección de red seguido de la wildcard correspondiente, ya sabemos que para permitir o denegar solo a uno usamos 'host' y para permitir o denegar a todo se usa 'any'.

#### ACL extended numbered

![a2](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgVueYpGfjUYiipMEio-hUBYjqtQ1jYJW06yCPACgDcZXf85jLLl9oxT3TPGUph69d2iyshUE79WWMNMHVbchAzPTXcfwZh3xw971Al7STYNtjZSoICG8VhV1uTJpTyiXyYhd-hJ0q6TfKTVT6BmLS9Su4EHRAIDb8_Fo47RxXD6arNUCAcXMcuiBy-CA/s1626/a2.png)

```html
R1(config)#access-list 101 deny tcp host 192.168.5.11 host 172.255.0.3 eq 80
R1(config)#access-list 101 permit ip any any
R1(config)#interface g0/0/1
R1(config-if)#ip access-group 101 in
R1(config-if)#do show access-lists 
Extended IP access list 101
    10 deny tcp host 192.168.5.11 host 172.255.0.3 eq www (12 match(es))
    20 permit ip any any (31 match(es))
```
Aquí si cambia en varias cosas primero se debe recordar que una ACL extendida se debe colocar lo más cerca del origen posible y en este caso es R1, también el rango de número disponibles para que puede ser ACL extendida, seguido de esto debemos poner lo que queremos hacer 'permitir' o 'denegar', ya sea un host o hacerlo para todos 'explicítamente' y al final el servicio al cual se niega o permite el acceso. Para asignar la ACL a la interfaz se coloca en 'in' para que sea inspeccionado antes de salir.

> El servidor tiene los servicios de DNS, HTTP, HTTPS, DHCP y FTP.

Básicamente la estructura de ACL extendida es la siguiente; número o nombre de ACL, 'permit' ó 'deny', protocolo ya sea (routing, tcp, udp, ip, icmp, gre, ahp), source origen (host-network_address-any), destino (host-network_address-any) y dependiendo el protocolo que especificamos aquí se pone más específicamente el servicio.

> La parte de origen y destino también la podemos tomar como un 'from-to'.

#### ACL extended named

![a3](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjKWtyV-gPjuo95bQv9j50ZhKC4195aX5cuumQ_ARM_WUUVB16tj9OWbz2cvaN0UjMOw7Wj9iVRYU_VJtJFe8kaydwm05s3PTknpBdK9wo5ujV_lp80QiaLXFw3O2VoFa8IIM4SMKFnm0stY0I6jkZTMjiCJPJNifjuTv6KIIF3Uf2F3dnWrmpvPl2-Hw/s1626/a3.png)

```html
R1(config)#ip access-list extended BLOCK-443
R1(config-ext-nacl)#permit tcp host 192.168.0.10 172.255.0.0 0.0.0.7 eq 443
R1(config-ext-nacl)#deny tcp 192.168.5.0 0.0.0.15 172.255.0.0 0.0.0.7 eq 443
R1(config-ext-nacl)#deny tcp 192.168.0.0 0.0.0.63 172.255.0.0 0.0.0.7 eq 443
R1(config-ext-nacl)#permit ip any any
R1(config-ext-nacl)#remark BLOQUEO WEB
R1(config-ext-nacl)#exit
R1(config)#interface g0/0/1
R1(config-if-range)#ip access-group BLOCK-443 in
R1(config-if-range)#end
R1#show access-lists
Extended IP access list 101
    10 deny tcp host 192.168.5.11 host 172.255.0.3 eq www
    20 permit ip any any
Extended IP access list BLOCK-443
    10 permit tcp host 192.168.0.10 172.255.0.0 0.0.0.7 eq 443 (7 match(es))
    20 deny tcp 192.168.5.0 0.0.0.15 172.255.0.0 0.0.0.7 eq 443 (53 match(es))
    30 deny tcp 192.168.0.0 0.0.0.63 172.255.0.0 0.0.0.7 eq 443 (16 match(es))
    40 permit ip any any (8 match(es))
```

En este escenario permitimos únicamente a un host de la red 192.168.0.0/26 para que acceda al puerto https seguro 443, y denegar el acceso a toda la red restante del host junto con la 192.168.5.0/28.

Primero se está permitiendo por tcp al único host de la red 192.168.0.0/26 para acceder a la 172.255.0.0/29 para que no de problemas debemos seguir este orden (primero el host luego la red del host), luego ya denegamos las redes junto con sus respectivas wildcards y el servicio, le agregamos un comentario y asignamos la ACL a la interfaz en 'in', por último vemos los resultados de la configuración aplicada los intentos realizados.

> Para que muestre los intentos es necesario antes en los equipos intentar acceder a cualquiera de estos servicios.

#### Eliminación de ACL

Para que una ACL ya tenga efecto debemos desasignarla a las interfaces y al dispositivo en general
```html
interface g0/0/0
    no ip access-group (número o nombre)
    no ip access-group 1
    no ip access-group BLOCKED
    exit
no ip access-list (tipo: standard o extended) (nombre)
no ip access-list extended BLOCKED
no access-list 101 deny tcp host 192.168.5.11 host 172.255.0.3 eq 80
no access-list 101 permit any any
```


## [Atrás](./)