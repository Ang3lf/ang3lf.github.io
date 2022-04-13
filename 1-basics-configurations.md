---
title: Configuraciones básicas
layout: default
---

*Basics configurations*, cuando estemos configurando un router o switch casi siempre es necesario tener unas configuraciones para el dispositivo, como contraseñas, hostnames, anuncios para los usuarios que la gestionan, entre otros.

Cuando estemos configurando un router por primera vez veremos algo similar a esto

```html
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#ip domain-name cisco.com
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exec-timeout 3 0
R1(config-line)#line vty 0 4
R1(config-line)#password Cisco1
R1(config-line)#login
R1(config-line)#exec-timeout 2 0
R1(config-line)#transport input ssh telnet
R1(config)#enable secret Cisco-level
R1(config)#service password-encryption
R1(config)#banner motd "Authorized personal only"
R1(config)#do copy runnning-config startup-config
```
De manera similar se haría en un switch cisco.

Las configuraciones en una cisco ASA son similares a las de un router, hay ligeros cambios en los comandos:
```html
ciscoasa>enable
Password:<Enter>
ciscoasa#configure terminal
ciscoasa(config)#hostname ASA
ASA(config)#enable password cisco
ASA(config)#domain-name contoso.com
ASA(config)#interface GigabitEthernet1/1
ASA(config-if)#ip address 209.165.200.1 255.255.255.248
ASA(config-if)#security-level 0-100
ASA(config-if)#nameif OUTSIDE/INSIDE
ASA(config-if)#no shutdown
```
> Nota: Se considera que una configuración básica de un dispositivo ASA es el direccionamiento para que pueda ser fácilmente gestionado, los parámetros que se indicaron en la interfaz como security-level indica que tan confiable es la interfaz, mientras mas alto el nivel es mas confiable la interfaz, y mientras más bajo menos confiable:
* security-level 0: nivel mas bajo que se asigna a una interfaz externa (WAN)
* security-level 100: nivel mas alto en una ASA que se asigna a una interfaz interna (LAN)
* security-levels 1-99: entre estos elejimos el nivel que mas nos convenga, por ejemplo con un nivel 50 prodríamos asignar a una DMZ 

#### Velocidad del reloj

Es la configuración que va en interfaces seriales que decide la velocidad a la que se envían los mensajes, velocidades de reloj altas > velocidades de transmisón más rápidas
```html
R1(config)#interface serial 0/1/0
R1(config-if)#clock rate 1200-4000000
```
Está configuración de clock rate debe ir en el router con serial principal, y los valores que tenemos para poner depende la versión de los dispositivos, pero en este caso es un router cisco ISR4321 y las interfaces son llamadas NIM-2T, los valores para poner va desde 1200 a 4000000.

* *DCE*: es el que controla paquetes
* *DTE*: es el que recibe paquetes

### Home router

Una configuración inportante en este tipo de router es cambiar la contraseña de administración del dispositivo, configurar SSID de los canales de transmisión con la máxima seguridad posible WPA2 o WP3 en algunos dispositivos nuevos. También es recomendable hacer segmentación de las redes para que personas ajenas no puedan tener acceso a la red principal.

[image1](./1.jpeg)

En esta ventana 'Setup' se configura los parámetros del direccionamiento IP en la LAN, el rango DHCP que lo distribuira el router mismo

[image2](./2.jpeg)

En la pestaña 'Wireless' configuramos las bandas de transmisión, los bandas varían depende al router

[image3](./3.jpeg)

En 'Wireless security' configuramos la seguridad inalámbrica, se recomienda usar WPA2 como seguridad porque es la mas fuerte hoy en día, la contraseña debe tener 8 caracteres mínimo

[image4](./4.jpeg)

Y en la pestaña de Administración cambiamos la contraseña por defecto, y ponemos una que sea más segura.

## [Atrás](./)