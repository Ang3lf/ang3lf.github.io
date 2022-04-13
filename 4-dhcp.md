---
title: DHCP
layout: default
---
*DHCP* es un protocolo de configuración dinámica de host, que consiste en asignar direcciones IP de forma dinámica a cada host en una red, este protocolo usa un modelo cliente/servidor con una serie de pasos:

* *DHCP/Discovery:* El cliente envía un paquete en difución 255.255.255.255 desde la dirección 0.0.0.0, con el objetivo de encontrar un servidor DHCP, si encutra a mas de uno se toma el primero que responda, de lo contrario solo recibe del único equipo DHCP.

* *DHCP/Offer:* Los equipos DHCP reciben el mensaje de difusión y envían un 'offer' al host que hizo la petición, y esa oferta lleva una dirección IP libre con máscara, y otra información como: gateway, dns, ip del servidor DHCP y ID del servidor. Un servidor DHCP escucha peticiones en el puerto 67/UDP.

* *DHCP/Request:* El cliente recibe las ofertas del servidor DHCP y "contacta" con el servidor que le envió esa oferta.

* *DHCP/ACK:* El servidor envia al cliente un acuse de recibo o un "recibido", y el cliente guarda localmente los datos proporcionados por el equipo DHCP como DNS, SMTP, POP3 y se conecta a la red.

> Configurar un servidor DHCP es mejor a comparación de hacerlo en forma estática, DHCP hace que sea menos propenso a errores.

Es posible configurar DHCP en routers empresariales, pero hay algo en contra y es que hacer este tipo de configuraciones sobrecarga la CPU del dispositivo.
```html
R1(config)#do show ip interface brief | begin GigabitEthernet0/0/2
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/2   192.168.100.1   YES manual up                    up 
Tunnel1                unassigned      YES unset  up                    down 
Vlan1                  unassigned      YES unset  administratively down down
R1(config)#ip dhcp pool NETWORK-1
R1(dhcp-config)#network 192.168.100.0 255.255.255.0
R1(dhcp-config)#default-router 192.168.100.1
R1(dhcp-config)#dns-server 192.168.100.254 
R1(dhcp-config)#domain-name contoso.com
R1(dhcp-config)#exit
R1(config)#ip dhcp excluded-address 192.168.100.1 192.168.100.11
R1(config)#ip dhcp excluded-address 192.168.100.254
R1(config)#service dhcp
```
```html
S1#configure terminal
S1(config)#interface vlan1
S1(config-if)#ip address dhcp
S1(config-if)#description NETWORK TO DHCP
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#do show ip interface brief | begin Vlan1
Interface              IP-Address      OK? Method Status                Protocol 
Vlan1                  192.168.100.12  YES DHCP   up                    up
```
> El cliente DHCP en este caso es un switch, en una computadora podemos ver la configuración aplicada con el comando 'ipconfig /all' windows y para linux y MAC-OS 'ifconfig'.

Las direcciones DHCP IP Helper son direcciones configuradas en una interfaz enrutada, como cualquier otra interfaz de enrutador que permite que ese dispositivo específico actúe como un "intermediario" que reenvía la solicitud DHCP BOOTP (Broadcast) que recibe en una interfaz al Servidor DHCP especificado por la dirección ip helper-address.
```html
S1(config)#interface vlan1
S1(config-if)#ip helper-address 192.168.100.1
S1(config-if)#end

Router(config)#interface g0/0/0
Router(config-if)#ip address dhcp 
Router(config)#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   192.168.100.14  YES DHCP   up                    up 
GigabitEthernet0/0/1   unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
Router(config)#do ping 192.168.100.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```
### DHCPv6
```html
R1(config)#interface g0/0/0
R1(config-if)#ipv6 address 2001:DB8:ACAD:A::1/64
R1(config-if)#ipv6 address FE80::1 link-local
R1(config-if)#ipv6 nd other-config-flag 
R1(config-if)#ipv6 dhcp server STATELESS
R1(config-if)#exit
R1(config)#do show ipv6 interface brief | begin GigabitEthernet0/0/2 
GigabitEthernet0/0/2       [up/up]
    FE80::1
    2001:DB8:ACAD:A::1
Serial0/1/0                [down/down]
    unassigned
Serial0/1/1                [administratively down/down]
    unassigned
Vlan1                      [administratively down/down]
    unassigned
R1(config)#ipv6 dhcp pool STATELESS
R1(config-dhcpv6)#address prefix 2001:db8:acad:a::/64
R1(config-dhcpv6)#dns-server 2001:db8:acad:a::5
R1(config-dhcpv6)#domain-name contoso2.com
R1(config-dhcpv6)#exit
R1(config)#ipv6 unicast-routing
```
> Para que un dispositivo de red sea un cliente DHCPv6 solo se pone en la interfaz ipv6 address dhcp, si es un router Cisco, primero se necesita activar ipv6 con sdm prefer dual-ipv4-and-ipv6 default.

_El parámetro ipv6 nd other-config-flag sirve para indicar que se puede obtener de forma automática información que no sean direcciones ipv6._

### Seguridad en DHCP
Un servidor o equipo DHCP regularmente puede sufrir spoofing attacks, y es posible cuando un cliente empieza con el proceso DHCPDisconver un atacante prodría hacerse pasar por el servidor DHCP y enviar todo el proceso al cliente, esta es una forma, otra forma que existe es que un atacante prodría haber ganado acceso a la red y hacerse con la dirección física (MAC) del verdadero servidor DHCP y hacer un reenvío de paquetes con el cliente desde el equipo atacante, poner un valor en /proc/net/sys/ipv4/ip_forward.

## [Atrás](./)