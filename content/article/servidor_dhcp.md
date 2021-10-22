---
#title: "Servidor DHCP"
date: 2021-10-22T19:09:13+02:00
draft: false
tags : ["Servicios"]
---

# **SERVIDOR DHCP**

**Tarea 1 (1 punto): Lee el documento Teoría: Servidor DHCP y explica el funcionamiento del servidor DHCP resumido en este gráfico.**

 Vamos a explicar un poco este gráfico, para que sepamos como funciona un Servidor DHCP:
!["IMAGEN"](/images/images_dhcp/IMAGEN.png)

 Tenemos cuatro mensajes de tipo broadcast, durante la obtención de una IP:
* DHCP DISCOVER --> Broadcast de cliente a servidores.
* DHCP OFFER --> Broadcast de servidores a cliente.
* DHCP REQUEST --> Broadcast de cliente a servidores.
* DHCP ACK --> Broadcast de servidor DHCP a cliente.

 Vamos a explicar todos sus estados, estados por los que pasa:

* INIT --> En este estado es en el que esta cuando se inicia

 Envia un DHCPDISCOVER al puerto del servidor (67/68). En el caso de que no exista un servidor DHCP, el router es el que se encarga, bueno en este caso es un agente DHCP relay que esta dentro del router.
Antes de enviar el mensaje se espera unos segundo para que no  colisione con los demás.

 Cuando el cliente esta en estado SELECTING, significa que va a recibir los mensajes DHCP OFFER del Servidor DHCP. Si ocurre el caso de que SELECTING recibe varios mensajes a la vez, el elegiría uno para seguir con el proceso. El cliente DHCP respondera este mensaje (DHCP REQUEST), y asi poder elegir un servidor DHCP, el cual contestara con un DHCP ACK. Este ultimo mensaje es el que contiene la configuración del cliente.

 El cliente tiene que enviar una petición ARP para que pueda comprobar que si ip sera solo para ese cliente y que no esta duplicada, si no esta duplicada, ya estaria en el estado BOUND, y tendría su configuración, pero en el caso de estar duplicado se envia un DHCP DECLINE para que así el cliente regrese al estado inicial (INIT) 

 El servidor asigna direcciones dentro de un rango prefijado, si en algún caso le asignamos manualmente a nuestro pc una ip ya repetida, lo que hará el servidor dhcp sera, el cliente solicitara y comprobará otras direcciones hasta que obtenga una que no este asignada a otro dispositivo.

**Preparación del escenario**

 Crea un escenario usando Vagrant que defina las siguientes máquinas:
* Servidor: Tiene dos tarjetas de red: una pública y una privada que se conectan a la red local.
* nodo_lan1: Un cliente conectado a la red local.

**Servidor DHCP**

 Instala un servidor dhcp en el ordenador “servidor” que de servicio a los ordenadores de red local, teniendo en cuenta que el tiempo de concesión sea 12 horas y que la red local tiene el direccionamiento 192.168.100.0/24.

 Yo en vez de la 192.168.100.0/24 usare la 192.168.200.0/24 porque mi maquina física tiene la 192.168.100.X

 Una vez que tengamos las maquinas instaladas con vagrant, accedemos a ellas por ssh de la siguiente forma:
* Accedemos al servidor:

>vagrant ssh nodo1

* Accedemos a la maquina cliente:

>vagrant ssh nodo2

 En el servidor instalamos el servidor dhcp:

>sudo apt-get install isc-dhcp-server

**Tarea 2: Entrega el fichero Vagrantfile que define el escenario.**

>Vagrant.configure("2") do |config|
>
>  config.vm.define :nodo1 do |nodo1|
>    nodo1.vm.box = "debian/buster64"
>    nodo1.vm.hostname = "servidor"
>    nodo1.vm.network "public_network"
>    nodo1.vm.network "private_network", ip: "192.168.200.6",
>      virtualbox__intnet: "red_local"
>  end
>  config.vm.define :nodo2 do |nodo2|
>    nodo2.vm.box = "debian/buster64"
>    nodo2.vm.hostname = "nodolan1"
>    nodo2.vm.network "private_network",
>      virtualbox__intnet: "red_local"
>  end
>end

**Tarea 3 (2 puntos): Muestra el fichero de configuración del servidor, la lista de concesiones, la modificación en la configuración que has hecho en el cliente para que tome la configuración de forma automática y muestra la salida del comando ` ip address`.**

 Desde el servidor vamos a editar el fichero /etc/network/interfaces. Editamos el eth2, ya que es donde tenemos la ip 192.168.200.6:

>auto eth2
>iface eth2 inet static
>      address 192.168.200.6
>      netmask 255.255.255.0
>      gateway 192.168.200.1
>      network 192.168.200.0
>      broadcast 192.168.200.255
>      dns-nameservers 8.8.8.8

 Reiniciamos la maquina del servidor:

>reboot

 A continuación editamos el fichero /etc/default/isc-dhcp-server. Aquí en este fichero en nuestro caso poner eth2 , donde pone INTERFACESv4:

>INTERFACESv4="eth2"

 Editamos el fichero /etc/dhcp/dhcpd.conf, en este fichero comentamos lo que viene descomentado y descomentamos lo siguiente, cambiando las partes necesarias:

>subnet 192.168.200.0 netmask 255.255.255.0 {
>  range 192.168.200.40 192.168.200.80;
>  option domain-name-servers 192.168.200.6;
>  option domain-name "servidor.local";
>  option routers 192.168.200.1;
>  option broadcast-address 192.168.200.255;
>  default-lease-time 43200;
>  max-lease-time 43200;
>}

 Reiniciamos el servicio:

>root@servidor:/home/vagrant# /etc/init.d/isc-dhcp-server restart
>[ ok ] Restarting isc-dhcp-server (via systemctl): isc-dhcp-server.servic.

 Comprobamos que esta funcionando correctamente:

>root@servidor:/home/vagrant# /etc/init.d/isc-dhcp-server restart
>[ ok ] Restarting isc-dhcp-server (via systemctl): isc-dhcp-server.servic.
>root@servidor:/home/vagrant# systemctl status isc-dhcp-server
>● isc-dhcp-server.service - LSB: DHCP server
>   Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
>  Active: active (running) since Fri 2021-01-29 17:51:38 GMT; 1min 19s ag
>     Docs: man:systemd-sysv-generator(8)
>  Process: 483 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, s
>    Tasks: 1 (limit: 544)
>   Memory: 5.1M
>  CGroup: /system.slice/isc-dhcp-server.service
>           └─496 /usr/sbin/dhcpd -4 -q -cf /etc/dhcp/dhcpd.conf eth2
>
>Jan 29 17:51:36 servidor systemd[1]: Starting LSB: DHCP server...
>Jan 29 17:51:36 servidor isc-dhcp-server[483]: Launching IPv4 server only.
>Jan 29 17:51:36 servidor dhcpd[496]: Wrote 0 leases to leases file.
>Jan 29 17:51:36 servidor dhcpd[496]: Server starting service.
>Jan 29 17:51:38 servidor isc-dhcp-server[483]: Starting ISC DHCPv4 server:
>Jan 29 17:51:38 servidor systemd[1]: Started LSB: DHCP server.

 Ahora pasamos a configurar el cliente, editaremos su fichero /etc/network/interfaces para que nuestra interfaz eth1 se conecta a nuestro servidor por dhcp y que así pueda darle una dirección.

>auto eth1
>iface eth1 inet dhcp

 Reiniciamos la maquina cliente:

>reboot

 Ahora tendriamos que pedirle al servidor que nos asigne una ip con el siguiente comando. Pero en nuestro caso a el reiniciar la maquina automáticamente nos asigna una ip:

>vagrant@nodolan1:~$ dhclient eth1

 Si hacemos un ip a, podemos comprobar que esta dentro del rango de ips:

>root@nodolan1:/home/vagrant# ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 82962sec preferred_lft 82962sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:69:0e:6e brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.40/24 brd 192.168.200.255 scope global dynamic eth1
>       valid_lft 39767sec preferred_lft 39767sec
>    inet6 fe80::a00:27ff:fe69:e6e/64 scope link 
>       valid_lft forever preferred_lft forever

 Comprobamos que entre el servidor y el cliente pueden hacerse ping mutuamente:

>root@servidor:/home/vagrant# ping 192.168.200.40
>PING 192.168.200.40 (192.168.200.40) 56(84) bytes of data.
>64 bytes from 192.168.200.40: icmp_seq=1 ttl=64 time=0.717 ms
>64 bytes from 192.168.200.40: icmp_seq=2 ttl=64 time=0.784 ms
>64 bytes from 192.168.200.40: icmp_seq=3 ttl=64 time=0.761 ms
>64 bytes from 192.168.200.40: icmp_seq=4 ttl=64 time=0.728 ms
>64 bytes from 192.168.200.40: icmp_seq=5 ttl=64 time=0.798 ms
>64 bytes from 192.168.200.40: icmp_seq=6 ttl=64 time=0.818 ms
>^C
>--- 192.168.200.40 ping statistics ---
>6 packets transmitted, 6 received, 0% packet loss, time 79ms
>rtt min/avg/max/mdev = 0.717/0.767/0.818/0.048 ms

>root@nodolan1:/home/vagrant# ping 192.168.200.6
>PING 192.168.200.6 (192.168.200.6) 56(84) bytes of data.
>64 bytes from 192.168.200.6: icmp_seq=1 ttl=64 time=0.753 ms
>64 bytes from 192.168.200.6: icmp_seq=2 ttl=64 time=0.972 ms
>64 bytes from 192.168.200.6: icmp_seq=3 ttl=64 time=0.903 ms
>64 bytes from 192.168.200.6: icmp_seq=4 ttl=64 time=0.841 ms
>64 bytes from 192.168.200.6: icmp_seq=5 ttl=64 time=0.948 ms
>^C
>--- 192.168.200.6 ping statistics ---
>5 packets transmitted, 5 received, 0% packet loss, time 12ms
>rtt min/avg/max/mdev = 0.753/0.883/0.972/0.083 ms

 Desde el servidor vamos a comprobar la lista de concesiones:

>root@servidor:/home/vagrant# tail -f /var/lib/dhcp/dhcpd.leases
>  cltt 6 2021/01/30 11:13:49;
>  binding state active;
>  next binding state free;
>  rewind binding state free;
>  hardware ethernet 08:00:27:69:0e:6e;
>  uid "\377'i\016n\000\001\000\001'\247\371\261\010\000'i\016n";
>  client-hostname "nodolan1";
>}

**Tarea 4 (1 puntos): Configura el servidor para que funcione como router y NAT, de esta forma los clientes tengan internet. Muestra las rutas por defecto del servidor y el cliente. Realiza una prueba de funcionamiento para comprobar que el cliente tiene acceso a internet (utiliza nombres, para comprobar que tiene resolución DNS).**

 Esta es nuestra ruta por defecto en el servidor, pero tenemos que cambiarla:

>root@servidor:/home/vagrant# ip r
>default via 10.0.2.2 dev eth0
>10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
>192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.40 
>192.168.200.0/24 dev eth2 proto kernel scope link src 192.168.200.6 

 La tenemos que cambiar porque queremos que tenga acceso desde la eth1:

>root@servidor:/home/vagrant# ip route change default via 192.168.100.1 dev eth1

 Comprobamos que se a cambiado la ruta y que tenemos conexión al exterior:

>root@servidor:/home/vagrant# ip r
>default via 192.168.100.1 dev eth1 
>10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
>192.168.100.0/24 dev eth1 proto kernel scope link src 192.168.100.51 
>192.168.200.0/24 dev eth2 proto kernel scope link src 192.168.200.6 

 Hacemos ping a www.marca.com y luego hacemos ping a otra pagina pero indicando la interfaz:

>root@servidor:/home/vagrant# ping www.marca.com
>PING unidadeditorial.map.fastly.net (199.232.193.50) 56(84) bytes of data.
>64 bytes from 199.232.193.50 (199.232.193.50): icmp_seq=1 ttl=55 time=11.4 ms
>64 bytes from 199.232.193.50 (199.232.193.50): icmp_seq=2 ttl=55 time=11.3 ms
>64 bytes from 199.232.193.50 (199.232.193.50): icmp_seq=3 ttl=55 time=11.4 ms
>64 bytes from 199.232.193.50 (199.232.193.50): icmp_seq=4 ttl=55 time=11.5 ms
>^C
>--- unidadeditorial.map.fastly.net ping statistics ---
>4 packets transmitted, 4 received, 0% packet loss, time 161ms
>rtt min/avg/max/mdev = 11.286/11.397/11.530/0.117 ms

>root@servidor:/home/vagrant# ping -I eth1 www.google.es
>PING www.google.es (216.58.209.67) from 192.168.100.51 eth1: 56(84) bytes of data.
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=1 ttl=117 time=13.1 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=2 ttl=117 time=12.1 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=3 ttl=117 time=12.1 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=4 ttl=117 time=13.1 ms
>^C
>--- www.google.es ping statistics ---
>4 packets transmitted, 4 received, 0% packet loss, time 201ms
>rtt min/avg/max/mdev = 12.070/12.594/13.096/0.503 ms

 Ahora para que nuestra maquina actúe como router tenemos que activar el "bit de fordward". Lo haremos permanentemente.

 Editamos el fichero /etc/sysctl.conf, y descomentamos la siguiente linea:

>net.ipv4.ip_forward=1

 En el fichero /etc/network/interfaces añadimos las instrucciones de ip tables:

>auto eth1
>iface eth1 inet dhcp
>    post-up ip route del default dev $IFACE || true
>
>up iptables -t nat -A POSTROUTING -o eth1 -s 192.168.100.0/24 -j MASQUERADE
>down iptables -t nat -D POSTROUTING -o eth1 -s 192.168.100.0/24 -j MASQUERADE

 Reiniciamos el servicio:

>root@servidor:/home/vagrant# /etc/init.d/isc-dhcp-server restart
>[ ok ] Restarting isc-dhcp-server (via systemctl): isc-dhcp-server.servic.

 Aqui podemos ver el direccionamiento de nuestras interfaces, la eth0 es por donde nos conectamos a virtualbox y a vagrant, la eth1 es la red publica, es la conexion entre nuestro servidor y nuestra maquina fisica y por ultimo la eth2 que es nuestra red privada por la cual nos comunicamos con el cliente:

* Hacemos un ip address desde el servidor:

>root@servidor:/home/vagrant# ip address
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 85370sec preferred_lft 85370sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:c7:4f:5d brd ff:ff:ff:ff:ff:ff
>    inet 192.168.100.51/24 brd 192.168.100.255 scope global dynamic eth1
>       valid_lft 85370sec preferred_lft 85370sec
>    inet6 fe80::a00:27ff:fec7:4f5d/64 scope link 
>       valid_lft forever preferred_lft forever
>4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:1d:00:22 brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.6/24 brd 192.168.200.255 scope global eth2
>       valid_lft forever preferred_lft forever
>    inet6 fe80::a00:27ff:fe1d:22/64 scope link 
>       valid_lft forever preferred_lft forever

* Hacemos un ip address desde el cliente:

>root@nodolan1:/home/vagrant# ip address
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 82264sec preferred_lft 82264sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:69:0e:6e brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.40/24 brd 192.168.200.255 scope global dynamic eth1
>       valid_lft 39069sec preferred_lft 39069sec
>    inet6 fe80::a00:27ff:fe69:e6e/64 scope link 
>       valid_lft forever preferred_lft forever

 Por ultimo, vamos a configurar el cliente. Modificamos nuestra ruta por defecto para que sea la eth1:

 Esta es su ruta por defecto:

>root@nodolan1:/home/vagrant# ip r
>default via 10.0.2.2 dev eth0 
>10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
>192.168.200.0/24 dev eth1 proto kernel scope link src 192.168.200.40 

 La cambiamos con el siguiente comando:

>root@nodolan1:/home/vagrant# ip route change default via 192.168.200.6 dev eth1

 Así quedaria la ruta una vez cambiada:

>root@nodolan1:/home/vagrant# ip r
>default via 192.168.200.6 dev eth1 
>10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
>192.168.200.0/24 dev eth1 proto kernel scope link src 192.168.200.40 

 Vamos a hacer la prueba, vamos a probar que nuestro router funciona. Hacemos ping desde nuestro cliente a la dirección 8.8.8.8:

>root@nodolan1:/home/vagrant# ping 8.8.8.8
>PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
>64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=54.1 ms
>64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=18.8 ms
>64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=12.9 ms
>64 bytes from 8.8.8.8: icmp_seq=4 ttl=116 time=13.7 ms
>64 bytes from 8.8.8.8: icmp_seq=5 ttl=116 time=12.6 ms
>64 bytes from 8.8.8.8: icmp_seq=6 ttl=116 time=12.7 ms
>^C
>--- 8.8.8.8 ping statistics ---
>6 packets transmitted, 6 received, 0% packet loss, time 14ms
>rtt min/avg/max/mdev = 12.552/20.774/54.069/15.044 ms

 Para la resolución de nombres tenemos que editar el fichero /etc/resolv.conf y le añadimos lo siguiente:

>domain servidor.local0
>search servidor.local0
>nameserver 192.168.200.6
>nameserver 8.8.8.8

 Ahora vamos a probar haciendo ping a google que resuelve nombres también:

>root@nodolan1:/home/vagrant# ping www.google.es
>PING www.google.es (216.58.209.67) 56(84) bytes of data.
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=1 ttl=116 time=13.4 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=2 ttl=116 time=15.0 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=3 ttl=116 time=14.6 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=4 ttl=116 time=12.8 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=5 ttl=116 time=12.8 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=6 ttl=116 time=13.9 ms
>64 bytes from waw02s06-in-f67.1e100.net (216.58.209.67): icmp_seq=7 ttl=116 time=13.3 ms
>^C
>--- www.google.es ping statistics ---
>7 packets transmitted, 7 received, 0% packet loss, time 20ms
>rtt min/avg/max/mdev = 12.767/13.679/15.010/0.812 ms

**Tarea 5 (1 punto): Realizar una captura, desde el servidor usando tcpdump, de los cuatro paquetes que corresponden a una concesión: DISCOVER, OFFER, REQUEST, ACK.**

 Instalamos tcpdump en nuestro servidor:

>sudo apt-get install tcpdump

 En el cliente borramos la ip para poder hacer la captura y volver a pedir otra ip:

>root@nodolan1:/home/vagrant# ip a del 192.168.200.40/24 dev eth1
>root@nodolan1:/home/vagrant# ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 79720sec preferred_lft 79720sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:69:0e:6e brd ff:ff:ff:ff:ff:ff
>   inet6 fe80::a00:27ff:fe69:e6e/64 scope link 
>       valid_lft forever preferred_lft forever

 Vamos a realizar una captura con la opción -vv es para que nos muestre la información mas detallada:

>root@servidor:/home/vagrant# tcpdump -vv -i eth2 -n
>tcpdump: listening on eth2, link-type EN10MB (Ethernet), capture size 262144 bytes
>13:11:55.834175 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
>    0.0.0.0.68 > 255.255.255.255.67: [udp sum ok] BOOTP/DHCP, Request from 08:00:27:69:0e:6e, length 300, xid 0xc7313b58, Flags [none] (0x0000)
>	  Client-Ethernet-Address 08:00:27:69:0e:6e
>	  Vendor-rfc1048 Extensions
>	    Magic Cookie 0x63825363
>	    DHCP-Message Option 53, length 1: Request
>	    Requested-IP Option 50, length 4: 192.168.200.41
>	    Hostname Option 12, length 8: "nodolan1"
>	    Parameter-Request Option 55, length 13: 
>	      Subnet-Mask, BR, Time-Zone, Default-Gateway
>	      Domain-Name, Domain-Name-Server, Option 119, Hostname
>	      Netbios-Name-Server, Netbios-Scope, MTU, Classless-Static-Route
>	      NTP
>13:11:55.834358 IP (tos 0x10, ttl 128, id 0, offset 0, flags [none], proto UDP (17), length 328)
>    192.168.200.6.67 > 192.168.200.41.68: [udp sum ok] BOOTP/DHCP, Reply, length 300, xid 0xc7313b58, Flags [none] (0x0000)
>	  Your-IP 192.168.200.41
>	  Client-Ethernet-Address 08:00:27:69:0e:6e
>	  Vendor-rfc1048 Extensions
>	    Magic Cookie 0x63825363
>	    DHCP-Message Option 53, length 1: ACK
>	    Server-ID Option 54, length 4: 192.168.200.6
>	    Lease-Time Option 51, length 4: 36090
>	    Subnet-Mask Option 1, length 4: 255.255.255.0
>	    BR Option 28, length 4: 192.168.200.255
>	    Default-Gateway Option 3, length 4: 192.168.200.1
>	    Domain-Name Option 15, length 15: "servidor.local0"
>	    Domain-Name-Server Option 6, length 4: 192.168.200.6

 Desde el cliente hemos pedido con dhclient eth1 que nos asigne una nueva ip:

>root@nodolan1:/home/vagrant# dhclient eth1
>root@nodolan1:/home/vagrant# ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 79307sec preferred_lft 79307sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:69:0e:6e brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.41/24 brd 192.168.200.255 scope global dynamic eth1
>       valid_lft 36088sec preferred_lft 36088sec
>    inet6 fe80::a00:27ff:fe69:e6e/64 scope link 
>       valid_lft forever preferred_lft forever

**Funcionamiento del dhcp**

 Vamos a comprobar que ocurre con la configuración de los clientes en determinadas circunstancias, para ello vamos a poner un tiempo de concesión muy bajo.

**Tarea 6 (1 punto): Los clientes toman una configuración, y a continuación apagamos el servidor dhcp. ¿qué ocurre con el cliente windows? ¿Y con el cliente linux?**

 Ya tenemos nuestra maquina windows en VirtualBox, ahora tenemos que cambiar su configuración para se conecte a la red interna llamada "red_local":
!["IMAGEN1"](/images/images_dhcp/IMAGEN-1.png)

 Iniciamos windows y comprobamos que el servidor DHCP le a asignado una ip dentro del rango que pusimos anteriormente:
!["IMAGEN2"](/images/images_dhcp/IMAGEN-2.png)

 En /etc/dhcp/dhcpd.conf cambiamos el tiempo de concesión, vamos a ponerle 1 minuto:

>subnet 192.168.200.0 netmask 255.255.255.0 {
>  range 192.168.200.40 192.168.200.80;
>  option domain-name-servers 192.168.200.6;
>  option domain-name "servidor.local0";
>  option routers 192.168.200.1;
>  option broadcast-address 192.168.200.255;
>  default-lease-time 60;
>  max-lease-time 43200;
>}

 Ahora vamos a apagar el servidor:

>root@servidor:/home/vagrant# poweroff
>Connection to 127.0.0.1 closed by remote host.
>Connection to 127.0.0.1 closed.
>mariajesus@debian:~/Vagrant$ 

 Ahora miramos en los clientes, pasado un minuto:
* Como podemos comprobar el linux no tiene ip:

>vagrant@nodolan1:~$ ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 86330sec preferred_lft 86330sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:69:0e:6e brd ff:ff:ff:ff:ff:ff
>    inet6 fe80::a00:27ff:fe69:e6e/64 scope link 
>       valid_lft forever preferred_lft forever

* Y en Windows obtiene una ip aletoria:
!["IMAGEN3"](/images/images_dhcp/IMAGEN-3.png)

**Tarea 7 (1 punto): Los clientes toman una configuración, y a continuación cambiamos la configuración del servidor dhcp (por ejemplo el rango). ¿qué ocurriría con un cliente windows? ¿Y con el cliente linux?**

 Lo primero que tenemos que hacer es editar el rango de ips en el fichero /etc/dhcp/dhcpd.conf:

>subnet 192.168.200.0 netmask 255.255.255.0 {
>  range 192.168.200.80 192.168.200.120;
>  option domain-name-servers 192.168.200.6;
>  option domain-name "servidor.local";
>  option routers 192.168.200.1;
>  option broadcast-address 192.168.200.255;
>  default-lease-time 43200;
>  max-lease-time 43200;
>}

 Ahora desde el cliente linux le pedimos que nos asigne una ip con dhclient eth1, y comprobamos con un ip a que nos asigna una ip dentro de el rango que le hemos puesto:

>root@nodolan1:/home/vagrant# dhclient eth1
>root@nodolan1:/home/vagrant# ip a>
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 86246sec preferred_lft 86246sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:d1:5b:0c brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.80/24 brd 192.168.200.255 scope global dynamic eth1
>       valid_lft 43199sec preferred_lft 43199sec
>    inet6 fe80::a00:27ff:fed1:5b0c/64 scope link 
>       valid_lft forever preferred_lft forever

Tambien comprobamos en el cliente windows que al hacer un ipconfig tiene otra ip y dentro del rango que hemos puesto:
!["IMAGEN4"](/images/images_dhcp/IMAGEN-4.png)

**Reservas**

 Crea una reserva para el que el cliente tome siempre la dirección 192.168.100.100.

 Yo en vez de la 192.168.100.100 usare la 192.168.200.100 porque mi maquina física tiene la 192.168.100.X

**Tarea 8 (1 puntos): Indica las modificaciones realizadas en los ficheros de configuración y entrega una comprobación de que el cliente ha tomado esa dirección.**

 En el fichero /etc/dhcp/dhcpd.conf añadimos lo siguiente:

>host nodolan1 {
>  hardware ethernet 08:00:27:7a:57:d9;
>  fixed-address 192.168.200.100;
>}

 Reiniciamos el servicio:

>root@servidor:/home/vagrant# systemctl restart isc-dhcp-server.service
>root@servidor:/home/vagrant# 

 Nos vamos a el cliente y reiniciamos. Comprobamos que le a asignado la ip que hemos reservado:

>root@nodolan1:/home/vagrant# ip a show dev eth1
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:7a:57:d9 brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.100/24 brd 192.168.200.255 scope global dynamic eth1
>       valid_lft 43175sec preferred_lft 43175sec
>    inet6 fe80::a00:27ff:fe7a:57d9/64 scope link 
>       valid_lft forever preferred_lft forever

**Uso de varios ámbitos**

 Modifica el escenario Vagrant para añadir una nueva red local y un nuevo nodo:
* Servidor: En el servidor hay que crear una nueva interfaz.
* nodo_lan2: Un cliente conectado a la segunda red local.

 Configura el servidor dhcp en el ordenador “servidor” para que de servicio a los ordenadores de la nueva red local, teniendo en cuenta que el tiempo de concesión sea 24 horas y que la red local tiene el direccionamiento 192.168.20.0/24.

**Tarea 9: Entrega el nuevo fichero Vagrantfile que define el escenario.**

>Vagrant.configure("2") do |config|
>
>  config.vm.define :nodo1 do |nodo1|
>    nodo1.vm.box = "debian/buster64"
>    nodo1.vm.hostname = "servidor"
>    nodo1.vm.network "public_network"
>    nodo1.vm.network "private_network", ip: "192.168.200.6",
>      virtualbox__intnet: "red_local1"
>    nodo1.vm.network :private_network, ip: "192.168.10.1", 
>      virtualbox__intnet: "red2"
>  end
>  config.vm.define :nodo2 do |nodo2|
>    nodo2.vm.box = "debian/buster64"
>    nodo2.vm.hostname = "nodolan1"
>    nodo2.vm.network "private_network",
>      virtualbox__intnet: "red_local1"
>  end
>  config.vm.define :nodo3 do |nodo3|
>    nodo3.vm.box="debian/buster64"
>    nodo3.vm.hostname="nodolan2"
>    nodo3.vm.network :private_network, 
>      virtualbox__intnet: "red_local2"
>  end
>end

**Tarea 10 (1 punto): Explica las modificaciones que has hecho en los distintos ficheros de configuración. Entrega las comprobaciones necesarias de que los dos ámbitos están funcionando.**

 Hacemos un ip a en el servidor y comprobamos que tenemos una nueva interfaz, la eth3:

>root@servidor:/home/vagrant# ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 85468sec preferred_lft 85468sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:d1:73:bb brd ff:ff:ff:ff:ff:ff
>    inet6 fe80::a00:27ff:fed1:73bb/64 scope link 
>       valid_lft forever preferred_lft forever
>4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8c:b5:50 brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.6/24 brd 192.168.200.255 scope global eth2
>       valid_lft forever preferred_lft forever
>    inet6 fe80::a00:27ff:fe8c:b550/64 scope link 
>       valid_lft forever preferred_lft forever
>5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:fd:e3:fb brd ff:ff:ff:ff:ff:ff
>    inet 192.168.10.1/24 brd 192.168.10.255 scope global eth3
>       valid_lft forever preferred_lft forever
>      inet6 fe80::a00:27ff:fefd:e3fb/64 scope link 
>       valid_lft forever preferred_lft forever

 Cambiamos la puerta de enlace:

>root@servidor:/home/vagrant# ip route change default via 192.168.100.1 dev eth1
>root@servidor:/home/vagrant# ip r
>default via 192.168.100.1 dev eth0 
>10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
>192.168.10.0/24 dev eth3 proto kernel scope link src 192.168.10.1 
>192.168.200.0/24 dev eth2 proto kernel scope link src 192.168.200.6 

 Modificamos el fichero /etc/default/isc-dhcp-server, le añadimos la nueva interfaz de red pero sin borrar la otra que teniamos:

>INTERFACESv4="eth2 eth3"

 En el fichero /etc/dhcp/dhcp.conf añadimos lo siguiente, que es la configuración de el otro cliente:

>subnet 192.168.10.0 netmask 255.255.255.0 {
>  range 192.168.10.80 192.168.200.120;
>  option domain-name-servers 8.8.8.8;
>  option domain-name "servidor.local2";
>  option routers 192.168.200.1;
>  options broadcast-address 192.168.10.255;
>  default-lease-time 86400;
>  max-lease-time 86400;
>}

 En el fichero /etc/network/interfaces y añadimos lo siguiente:

>auto eth3
>iface eth3 inet static
>      address 192.168.10.1
>      netmask 255.255.255.0
>      gateway 192.168.10.1
>      network 192.168.10.0
>      broadcast 192.168.10.255
>      dns-nameservers 8.8.8.8

 Reiniciamos el servicio:

>root@servidor:/home/vagrant# systemctl restart isc-dhcp-server.service

 Activamos el "bit de forward", esto lo hacemos editando el fichero /etc/sysctl.conf:

>net.ipv4.ip_forward=1

 En el fichero /etc/network/interfaces del cliente cambiamos lo siguiente:

>auto eth1
>iface eth1 inet dhcp

 Como podemos comprobar nuestros clientes al iniciarlos tienen las ips dentro del rango:

* En la maquina llamada "nodolan1" nos asigna la 192.168.200.100 por la reserva que hicimos anteriormente:

>root@nodolan1:/home/vagrant# ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>      valid_lft 83724sec preferred_lft 83724sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
>    link/ether 08:00:27:d3:f7:c2 brd ff:ff:ff:ff:ff:ff
>    inet 192.168.200.100/24 scope global eth1
>       valid_lft forever preferred_lft forever

* Y en la maquina llamada "nodolan2" nos asigna una ip dentro del rango:

>root@nodolan2:/home/vagrant# ip a
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
>    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
>    inet 127.0.0.1/8 scope host lo
>       valid_lft forever preferred_lft forever
>    inet6 ::1/128 scope host 
>       valid_lft forever preferred_lft forever
>2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
>    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
>    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
>       valid_lft 83468sec preferred_lft 83468sec
>    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
>       valid_lft forever preferred_lft forever
>3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
>    link/ether 08:00:27:b5:91:bc brd ff:ff:ff:ff:ff:ff
>    inet 192.168.10.80/24 scope global eth1
>       valid_lft forever preferred_lft forever

**Tarea 11 (1 punto): Realiza las modificaciones necesarias para que los cliente de la segunda red local tengan acceso a internet. Entrega las comprobaciones necesarias.**

 Añadimos una nueva regla iptables:

>iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth1 -j MASQUERADE

 Cambiamos la puerta de enlace del cliente:

>root@nodolan2:/home/vagrant# ip route change default via 192.168.10.1 dev eth1

 Ya tenemos internet en nuestro cliente llamado "nodolan2".
