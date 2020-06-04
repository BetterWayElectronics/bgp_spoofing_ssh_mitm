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

I then modified my default SSH port to 1337 and enabled this port in my nftables.

I reset the BGP and Zebra services and the `ip route` now appeared as such:

![](https://i.imgur.com/qu8rOE9.png)

This meant that I was now intercepting all communication to and from 10.45.0.0/25.

![](https://i.imgur.com/91MSMEk.png)

If I looked at the routing from another machine, it is again confirmed.

## SSH MiTM

I decided against an interactive honeypot as I simply wanted their SSH passwords, they would assume they were already hacked or there was an issue with their password/session in this scenario - but this is only given if I kept the interception going for a short period.

I uploaded the ssh-honeypot to the server and installed it with the following commands

* apt-get install clang make libssh-dev libjson-c-dev
* ssh-keygen -t rsa -f ./ssh-honeypot.rsa
* bin/ssh-honeypot –r ./ssh-honeypot.rsa

I then ran the honeypot with the command `bin/ssh-honeypot -h` and it was done.
<br>If I attempted to connect to 10.45.0.1 I would get the following result:

![](https://i.imgur.com/VEwb4Os.png)

A success! I could now from this point monitor any unencrypted data from 10.45.0.1 with the command `tcpdump –I ens192 –X src 10.45.0.1`. Though in this environment there was no opportunity to do so.

I could also add every single IP in the network to my Zebra and BGP configuration. This would force all IPs to connect through me. 

![](https://i.imgur.com/Oiqx2f2.png)



