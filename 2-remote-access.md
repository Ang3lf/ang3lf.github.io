---
title: Configuración acceso remoto
layout: default
---

### Acceso remoto:

Es la capacidad de acceder a una computadora u otro dispositivo desde un dispositvo, en cualquier momento y lugar, con algunas configuraciones podemos hacer posible esta forma de acceder a un dispositivo y tomar el control de los archivos, aplicaciones entre otras.

### Requesitos:

* Dispositivo con una dirección IP asignada, dinámica o estática
* Servicio corriendo
* Reglas para que permitan el tráfico

### Servicios:

Existen varios protocolos de acceso remoto, en los dispositivos cisco podemos configurar 2 que son Telnet y SSH, ambos son protocolos de acceso remoto.

#### Puertos:

> SSH trabaja en el puerto 22/tcp
> Telnet trabaja en el puerto 23/tcp

#### Diferencias:

* *SSH:* Permite acceso remoto de forma segura a otra dispositivo, ya que este cifra la información en todo momento y asegura la integridad de los datos.
* *Telnet:* Permite acceso remoto pero a diferencia de SSH este no cifra la información de los dispositvos que esten conectados, este protocolo manda toda la información en texto plano, lo cual lo hace peligroso para los usarios debido a que la información queda expuesta.

### SSH

Con la dirección IP que tenga el equipo de red podremos empezar a configurar en las líneas o inteefaces del equipo:
```html
R1> enable
R1#configure terminal
R1(config)#interface gigabitEthernet 0/0/0
R1(config-if)#ip address 10.129.160.1 255.255.255.0
R1(config-if)#description NETWORK-1
R1(config-if)#no shutdown
R1(config)#ip domain-name cisco.com
R1(config)#crypto key generate rsa
The name for the keys will be: R1.cisco.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

R1(config)#ip ssh version 2
R1(config)#username admin privilege 15 password Str0ngP@ssw0rD
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#exec-timeout 60
R1(config-line)#transport input ssh
R1(config-line)#login local
```
Podemos verificar las sesiones que tiene abiertas el equipo con el siguiente comando:

```html
R1#show ssh
Connection      Version Mode Encryption  Hmac State             Username
3               1.99    IN   aes128-cbc      hmac-sha1     Session Started         admin
3               1.99    OUT  aes128-cbc      hmac-sha1     Session Started   admin
%No SSHv1 server connections running.
R1#copy running-config startup-config
```

### Telnet

Teniendo ya configurada en la interfaz en una dirección IP podemos empezar a configurar telnet para acceso remoto:
```html
Router(config)#hostname R1
R1(config)#in g0/0/0
R1(config-if)#ip add 192.168.1.1 255.255.255.0
R1(config-if)#des
R1(config-if)#description LOCAL-1
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#line vty 0 4
R1(config-line)#login local
R1(config-line)#password cisco
R1(config-line)#exec-timeout 15 0
R1(config-line)#transport input telnet 
R1(config)#username bob password secret
```

Para poder acceder, por ejemplo desde una computadora en la terminal podemos poner para SSH:
```bash
ssh -l admin 10.129.160.1
[password]: cisco
ssh admin@10.129.160.1
```

Para Telnet:
```bash
telnet 192.168.1.1
[usernmae]: bob
[password]: secret
```
Para un switch la configuración es algo similar, la diferencia es que el switch debemos configurar la IP en las vlans, si contamos con segmentación de vlans en el switch, debemos tener en cuenta cual es la que se configura para dar  un correcto acceso.

> Con Wireshark o Tshark podemos ver el tráfico de la sesión si filtramos por 'ssh' o 'telnet', y veremos lo comentado anteriormente, ssh cifra la información que fluye en ambos equipos mientras que telnet no.

## [Atrás](./)