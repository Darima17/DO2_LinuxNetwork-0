# DO2 Linux Network
## Part 1. ipcalc tool
### 1.1. Networks and Masks
#### 1) Network address of 192.167.38.54/13 
![1.1.1](../misc/images/report_img/1.1.1.png)
#### 2) Conversion of the mask 255.255.255.0 to prefix and binary, /15 to normal and binary, 11111111.11111111.11111111.11110000 to normal and prefix
![1.1.2](../misc/images/report_img/1.1.2.png)
#### 3) Minimum and maximum host in 12.167.38.4 network with masks:
* /8

![1.1.3.1](../misc/images/report_img/1.1.3.1.png)

* 11111111.11111111.00000000.00000000

![1.1.3.2](../misc/images/report_img/1.1.3.2.png)

* 255.255.254.0

![1.1.3.3](../misc/images/report_img/1.1.3.3.png)

* /4

![1.1.3.4](../misc/images/report_img/1.1.3.4.png)

### 1.2. localhost
#### Define and write in the report whether an application running on localhost can be accessed with the following IPs:
IP localhosts on ranges [127.0.0.1 - 127.255.255.254]
|IP|ACCESS|
|--|----|
|194.34.23.100|FALSE|
|127.0.0.2|TRUE|
|127.1.0.1|TRUE|
|128.0.0.1|FALSE|

### 1.3. Network ranges and segments
#### Define and write in a report:
#### 1) which of the listed IPs can be used as public and which only as private: 10.0.0.45, 134.43.0.2, 192.168.4.2, 172.20.250.4, 172.0.2.1, 192.172.0.1, 172.68.0.2, 172.16.255.255, 10.10.10.10, 192.169.168.1
|Private range|10.0.0.0 – 10.255.255.255|172.16.0.0 – 172.31.255.255|192.168.0.0 – 192.168.255.255|
|-|-|-|-|

[Difference private and public IP](https://www.geeksforgeeks.org/difference-between-private-and-public-ip-addresses/)
|PUBLIC|PRIVATE|
|------|-------|
|134.43.0.2|10.0.0.45|
|172.0.2.1|192.168.4.2|
|192.172.0.1|172.20.250.4|
|172.68.0.2|172.16.255.255|
|10.10.10.10|192.169.168.1|

#### 2) which of the listed gateway IP addresses are possible for 10.10.0.0/18 network: 10.0.0.1, 10.10.0.2, 10.10.10.10, 10.10.100.1, 10.10.1.255
##### Access IP range: 10.10.0.1 - 10.10.63.254
![1.1.4.2](../misc/images/report_img/1.1.4.2.png)

|IP|ACCESS|
|------|-------|
|10.0.0.1|FALSE|
|10.10.0.2|TRUE|
|10.10.10.10|TRUE|
|10.10.100.1|FALSE|
|10.10.1.255|TRUE|

## Part 2. Static routing between two machines
#### Start two virtual machines (hereafter -- ws1 and ws2. View existing network interfaces with the `ip a` command
![2.0.1](../misc/images/report_img/2.0.1.png)

* enp0s3 name interface for ws1 and ws2

#### Describe the network interface corresponding to the internal network on both machines and set the following addresses and masks: ws1 - 192.168.100.10, mask /16, ws2 - 172.24.116.8, mask /12

`sudo nano etc/netplan/00-installer-config.yaml`
* ws1:

![2.0.2](../misc/images/report_img/2.0.2.png)

* ws2:

![2.0.3](../misc/images/report_img/2.0.3.png)

`sudo netplan apply`

* ws1:

![2.0.4](../misc/images/report_img/2.0.4.png)

* ws2:

![2.0.5](../misc/images/report_img/2.0.5.png)

### 2.1. Adding a static route manually

* ws1:

`sudo ip r add 172.24.116.8 dev enp0s3`
`sudo ping 192.168.100.10`

![2.1.1](../misc/images/report_img/2.1.1.png)

* ws2:

`sudo ip r add 192.168.100.10 dev enp0s3`
`sudo ping 172.24.116.8`

![2.1.2](../misc/images/report_img/2.1.2.png)

### 2.2. Adding a static route with saving

[ROUTES](https://linuxconfig.org/how-to-add-static-route-with-netplan-on-ubuntu-20-04-focal-fossa-linux)

`sudo nano etc/netplan/00-installer-config.yaml`

* ws1:

![2.2.1](../misc/images/report_img/2.2.1.png)

* ws2:

![2.2.2](../misc/images/report_img/2.2.2.png)

* ws1 :
`sudo netplan apply`
`ping 172.24.116.8`

![2.2.3](../misc/images/report_img/2.2.3.png)

* ws2 : `sudo ip route flush table main;` `sudo ip route flush cache;` `sudo netplan apply;` `ping 192.168.100.10;` (`ip route; ` commands for apply settings without restart network service)

![2.2.4](../misc/images/report_img/2.2.4.png)

## Part 3. iperf3 utility

### 3.1. Connection speed

8 Mbps = 1 MB/s, 100 MB/s = 819200 Kbps, 1 Gbps = 1024 Mbps

### 3.2. iperf3 utility

* ws1: `sudo iperf3 -c 172.24.166.8 -p 5201`

![3.2.1](../misc/images/report_img/3.2.1.png)

* ws2: `sudo iperf3 -s`

![3.2.2](../misc/images/report_img/3.2.2.png)

* ws2: `sudo iperf3 -c 192.168.100.10 -f M`

![3.2.3](../misc/images/report_img/3.2.3.png)

* ws1: `sudo iperf3 -s`

![3.2.4](../misc/images/report_img/3.2.4.png)

## Part 4. Network firewall

`sudo ufw disable` (if firewall active)

![4.0](../misc/images/report_img/4.0.png)


### 4.1. iptables utility
Create a /etc/firewall.sh file simulating the firewall on ws1 and ws2:
##### 3) open access on machines for port 22 (ssh) and port 80 (http)
```shell
#!/bin/sh

# Deleting all the rules in the "filter" table (default).
iptables -F
iptables –X
# open access on machines for port 22 (ssh)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
# and port 80 (http)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
```
ws1:
```shell
# iptables -{A|I} {INPUT|OUTPUT} -p icmp --icmp-type {echo-reply|echo-request} -j {ACCEPT|REJECT|DROP}
# on ws1 apply a strategy where a deny rule is written at the beginning
iptables -I OUTPUT -p icmp --icmp-type echo-reply -j DROP
# and an allow rule is written at the end (this applies to points 4 and 5)
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
/sbin/iptables-save
```
![4.1.1](../misc/images/report_img/4.1.1.png)


ws2:
```shell
# iptables -{A|I} {INPUT|OUTPUT} -p icmp --icmp-type {echo-reply|echo-request} -j {ACCEPT|REJECT|DROP}
# on ws2 apply a strategy where an allow rule is written at the beginning 
iptables -I OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
# and a deny rule is written at the end (this applies to points 4 and 5)
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j DROP
/sbin/iptables-save
```
![4.1.3](../misc/images/report_img/4.1.3.png)

##### Run the files on both machines with `chmod +x /etc/firewall.sh` and `/etc/firewall.sh` commands.
##### ws1:
![4.1.2](../misc/images/report_img/4.1.2.png)
##### ws2:
![4.1.4](../misc/images/report_img/4.1.4.png)

#### If a rule does not match the packet, the packet is passed to the next rule. If a rule does match the packet, the rule takes the action indicated by the target/verdict, which may result in the packet being allowed to continue along the chain or it may not. On ws1 first rule `DROP` break the chain also ws2 first rule `ACCEPT` break the chain but ping paсkets go to ws2.

### SSH

#### Work in VM from VirtualBox very difficult and now we connect second adapter (NAT) and connect VM to host machine and internet.

1) Enable second network adapter NAT:

![0.0](../misc/images/report_img/0.0.png)

2) Start VM:

`ip a`

![0.1](../misc/images/report_img/0.1.png)


##### `enp0s8` interface second adapter but now is not working. Go to netplan setting and set parameters for this interface:

![0.2](../misc/images/report_img/0.2.png)

##### Apply command: `sudo netplan apply`

![0.3](../misc/images/report_img/0.3.png)

##### Now we see ip address `enp0s8`. But we can't ssh connect directly to this ip. Solution is forwarding port `22` on ip `10.0.3.15` to port `2223` on `127.0.0.1`

![0.4](../misc/images/report_img/0.4.png)

`sudo reboot` - executing command 

##### Try to connect ws2 from ssh `sudo ssh -p 2223 janiecee@127.0.0.1`

![0.5](../misc/images/report_img/0.5.png)

#### 4.2. **nmap** utility

`apt install nmap`
##### Use **ping** command to find a machine which is not pinged, then use **nmap** utility to show that the machine host is up
*Check: nmap output should say: `Host is up`*.

![4.2.1](../misc/images/report_img/4.2.1.png)

- Add screenshots with the call and output of the **ping** and **nmap** commands to the report.

##### Save dumps of the virtual machine images

## Part 5. Static network routing

`-` So far we have only connected two machines, but now it's time for static routing of the whole network.


Network: \
<img src="../misc/images/part5_network.png" alt="part5_network" width="800"/>

##### Start five virtual machines (3 workstations (ws11, ws21, ws22) and 2 routers (r1, r2))

Minimal config for VM: \
<img src="../misc/images/report_img/5.0.1.png" alt="part5_network" width="800"/>

Clone VM: \
<img src="../misc/images/report_img/5.0.png"/>

### !! BEFORE EXECUTING MAKE SURE DIFFERENCE VM'S MAC-ADDRESSES

#### 5.1. Configuration of machine addresses
##### Set up the machine configurations in *etc/netplan/00-installer-config.yaml* according to the network in the picture.
*etc/netplan/00-installer-config.yaml*
#### ws11:
![5.1.1](../misc/images/report_img/5.1.1.png)
#### ws21:
![5.1.2](../misc/images/report_img/5.1.2.png)
#### ws22:
![5.1.3](../misc/images/report_img/5.1.3.png)
#### r1:
![5.1.4](../misc/images/report_img/5.1.4.png)
#### r2:
![5.1.5](../misc/images/report_img/5.1.5.png)
##### Restart the network service. If there are no errors, check that the machine address is correct with the `ip -4 a`command. Also ping ws22 from ws21. Similarly ping r1 from ws11.
#### ws11:
![5.1.6](../misc/images/report_img/5.1.6.png)
#### ws21:
![5.1.7](../misc/images/report_img/5.1.7.png)
#### ws22:
![5.1.8](../misc/images/report_img/5.1.8.png)
#### r1:
![5.1.9](../misc/images/report_img/5.1.9.png)
#### r2:
![5.1.10](../misc/images/report_img/5.1.10.png)

#### 5.2. Enabling IP forwarding.
##### To enable IP forwarding, run the following command on the routers:
`sysctl -w net.ipv4.ip_forward=1`
* r1:

![5.2.1](../misc/images/report_img/5.2.1.png)

* r2:

![5.2.2](../misc/images/report_img/5.2.2.png)

*With this approach, the forwarding will not work after the system is rebooted.*

##### Open */etc/sysctl.conf* file and add the following line:
`net.ipv4.ip_forward = 1`
* r1:

![5.2.3](../misc/images/report_img/5.2.3.png)

* r2:

![5.2.4](../misc/images/report_img/5.2.4.png)

*With this approach, IP forwarding is enabled permanently.*
When `ip_forward=0`, it thinks: "I don't know why this got sent to me and I don't really care. To the trash it goes!"

With `ip_forward=1`, "Hmm, this is not for me. But I know where the recipient is, so I'll just resend it with the correct MAC address."

#### 5.3. Default route configuration

##### Configure the default route (gateway) for the workstations. To do this, add `default` before the router's IP in the configuration file
#### ws11:
![5.2.5](../misc/images/report_img/5.2.5.png)
#### ws21:
![5.2.6](../misc/images/report_img/5.2.6.png)
#### ws22:
![5.2.7](../misc/images/report_img/5.2.7.png)
##### * `gateway4:` is old and now not use (but work)
##### Call `ip r` and show that a route is added to the routing table
#### ws11:
![5.2.8](../misc/images/report_img/5.2.8.png)
#### ws21:
![5.2.9](../misc/images/report_img/5.2.9.png)
#### ws22:
![5.2.10](../misc/images/report_img/5.2.10.png)
##### Ping r2 router from ws11 and show on r2 that the ping is reaching.

#### ws11:
![5.2.11](../misc/images/report_img/5.2.11.png)
##### To do this, use the `tcpdump -tn -i eth1 icmp` filtered `icmp` because ssh connect send any more packets
#### r2:
![5.2.12](../misc/images/report_img/5.2.12.png)

#### 5.4. Adding static routes
##### Add static routes to r1 and r2 in configuration file. Here is an example for r1 route to 10.20.0.0/26:
```shell
# Add description to the end of the eth1 network interface:
- to: 10.20.0.0
  via: 10.100.0.12
```
#### r1:
![5.4.1](../misc/images/report_img/5.4.1.png)
#### r2:
![5.4.2](../misc/images/report_img/5.4.2.png)

##### Call `ip r` and show route tables on both routers. Here is an example of the r1 table:
```
10.100.0.0/16 dev eth1 proto kernel scope link src 10.100.0.11
10.20.0.0/26 via 10.100.0.12 dev eth1
10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.1
```
#### Output on revers sequence and i don't know why:
#### r1:
![5.4.3](../misc/images/report_img/5.4.3.png)
#### r2:
![5.4.4](../misc/images/report_img/5.4.4.png)
##### Run `ip r list 10.10.0.0/[netmask]` and `ip r list 0.0.0.0/0` commands on ws11.
![5.4.5](../misc/images/report_img/5.4.5.png)

- Explain in the report why a different route other than 0.0.0.0/0 had been selected for 10.10.0.0/18 although it could be the default route: 
Because a more optimal route on this network based on system metrics.

#### 5.5. Making a router list

##### Run the `tcpdump -tnv -i eth0` dump command on r1
![5.5.1](../misc/images/report_img/5.5.2.png)
![5.5.1](../misc/images/report_img/5.5.3.png)
![5.5.1](../misc/images/report_img/5.5.4.png)
![5.5.1](../misc/images/report_img/5.5.5.png)
![5.5.1](../misc/images/report_img/5.5.6.png)

**traceroute** utility output after adding a gateway to list routers in the path from ws11 to ws21

![5.5.7](../misc/images/report_img/5.5.7.png)

- Based on the output of the dump on r1, explain in the report how path construction works using **traceroute**: 

`-t` don't print a timestamp on each dump line.
`-n` don't  convert  addresses  (i.e.,  host addresses, port numbers, etc.) to names.
`-v` when parsing and printing, produce (slightly more) verbose  output.
`-i` listen on interface.

Traceroute send packets with `TTL = 1` and each host doing `-=TTL`. Hence my gateway server will send me back a TTL Time exceeded message. On receiving this TTL Time exceeded message, my traceroute program will come to know the source address and other details about the first hop (Which is my gateway server).
Next packet with `TTL = 2` This is because my gateway router will reduce it by 1 and then forwards that same packet which send to the next hop/router (the packet send by my gateway to its next hop will have a TTL value of 1). 
On receiving UDP packet, the next hop to my gateway server will once again reduce it to 1 which means now the TTL has once again become 0. Hence it will send me back a ICMP Time exceeded message with its source address, and also the first 28 byte header of the packet which i send.
On receiving that message of TTL Time Exceeded, my traceroute program will come to know about that hop/routers IP address and it will show that on my screen.

#### 5.6. Using **ICMP** protocol in routing
##### Run on r1 network traffic capture going through eth0 with the
#####  `tcpdump -n -i eth0 icmp`
![5.6.2](../misc/images/report_img/5.6.2.png)
##### Ping a non-existent IP (e.g. *10.30.0.111*) from ws11 with the
##### `ping -c 1 10.30.0.111`
![5.6.1](../misc/images/report_img/5.6.1.png)


##### Save dumps of the virtual machine images
1) Get root
`sudo su`

2) List of VM for registration
`vboxmanage list vms`

3) Registration VM's
`VBoxManage registervm '~/VirtualBox VMs/ws11/ws11.vbox' '~/VirtualBox VMs/ws21/ws21.vbox' '~/VirtualBox VMs/ws22/ws22.vbox' '~/VirtualBox VMs/r1/r1.vbox' '~/VirtualBox VMs/r2/r2.vbox'`

4) Start VM's
`vboxmanage startvm ws11 ws21 ws22 r1 r2`

5) Save dump's
 `cd ../ws21/; vboxmanage snapshot ws21 take ws21_part5; cd ../ws22/; vboxmanage snapshot ws22 take ws22_part5; cd ../ws11/; vboxmanage snapshot ws11 take ws11_part5; cd ../r1/; vboxmanage snapshot r1 take r1_part5; cd ../r2/; vboxmanage snapshot r2 take r2_part5;`
 
* Start VM's without GUI: `vboxmanage startvm ws11 ws21 ws22 r1 r2 --type headless`

## Part 6. Dynamic IP configuration using **DHCP**

`-` Our next step is to learn more about **DHCP** service, which you already know.

##### First step install DHCP server:

`apt install isc-dhcp-server`

##### If you have more one interface (example first enp0s3 - internet connection):

*/etc/default/isc-dhcp-server* open on editor and change:

`INTERFACESv4="enp0s8"`, `INTERFACESv6="enp0s8"`

##### For r2, configure the **DHCP** service in the */etc/dhcp/dhcpd.conf* file:

##### 1) specify the default router address, DNS-server and internal network address. Here is an example of a file for r2:
```shell
subnet 10.100.0.0 netmask 255.255.0.0 {}

subnet 10.20.0.0 netmask 255.255.255.192
{
    range 10.20.0.2 10.20.0.50;
    option routers 10.20.0.1;
    option domain-name-servers 10.20.0.1;
}
```
![6.2.1](../misc/images/report_img/6.2.1.png)
##### 2) Write `nameserver 8.8.8.8.` in a *resolv.conf* file
![6.2.2](../misc/images/report_img/6.2.2.png)
##### Restart the **DHCP** service with `systemctl restart isc-dhcp-server`.
![6.2.3](../misc/images/report_img/6.2.3.png)
##### Reboot the ws21 machine with `reboot` and show with `ip a` that it has got an address.
![6.2.4](../misc/images/report_img/6.2.4.png)
##### Also ping ws22 from ws21.
![6.2.5](../misc/images/report_img/6.2.5.png)

##### Specify MAC address at ws11 by adding to *etc/netplan/00-installer-config.yaml*:
`macaddress: 10:10:10:10:10:BA`, `dhcp4: true`

##### ws11:
![6.2.6](../misc/images/report_img/6.2.6.png)

##### Сonfigure r1 the same way as r2, but make the assignment of addresses strictly linked to the MAC-address (ws11).
![6.2.7](../misc/images/report_img/6.2.7.png)

##### Run the same tests
##### Restart the **DHCP** service with `systemctl restart isc-dhcp-server`.
![6.2.8](../misc/images/report_img/6.2.8.png)

##### Reboot the ws11 machine with `reboot` and show with `ip a` that it has got an address:
![6.2.9](../misc/images/report_img/6.2.9.png)
##### Also ping r1 from ws1:
![6.2.10](../misc/images/report_img/6.2.10.png)

##### Request ip address update from ws21:
![6.2.11](../misc/images/report_img/6.2.11.png)
##### ip before:
![6.2.12](../misc/images/report_img/6.2.12.png)
##### and after command `dhclient -v`:
![6.2.13](../misc/images/report_img/6.2.13.png)

- Describe in the report what **DHCP** server options were used in this point.

* `option routers [10.0.0.0,...]` the routers option specifies a list of IP addresses for routers on the client's subnet. Routers should be listed in order of preference

* `option domain-name-servers [10.0.0.0, ...]` domain-name-servers option specifies a list of Domain Name System (STD 13, RFC 1035) name servers available to the client. Servers should be listed in order of preference.

##### Save dumps of virtual machine images
`cd ../ws21/; vboxmanage snapshot ws21 take ws21_part6; cd ../ws22/; vboxmanage snapshot ws22 take ws22_part6; cd ../ws11/; vboxmanage snapshot ws11 take ws11_part6; cd ../r1/; vboxmanage snapshot r1 take r1_part6; cd ../r2/; vboxmanage snapshot r2 take r2_part6;`

## Part 7. **NAT**

##### In */etc/apache2/ports.conf* file change the line `Listen 80` to `Listen 0.0.0.0:80`on ws22 and r1, i.e. make the Apache2 server public
##### ws22:
![7.1.1](../misc/images/report_img/7.1.1.png)
##### r1:
![7.1.2](../misc/images/report_img/7.1.2.png)
##### Start the Apache web server with `service apache2 start` command on ws22:
![7.1.3](../misc/images/report_img/7.1.3.png)
##### and r1:
![7.1.4](../misc/images/report_img/7.1.4.png)

##### Add the following rules to the firewall, created similarly to the firewall from Part 4, on r2:
##### 1) Delete rules in the filter table - `iptables -F`
##### 2) Delete rules in the "NAT" table - `iptables -F -t nat`
##### 3) Drop all routed packets - `iptables --policy FORWARD DROP`
![7.1.5](../misc/images/report_img/7.1.5.png)
##### Run the file as in Part 4
![7.1.6](../misc/images/report_img/7.1.6.png)
##### Check the connection between ws22 and r1 with the `ping` command
*When running the file with these rules, ws22 should not ping from r1*

![7.1.7](../misc/images/report_img/7.1.7.png)
##### Add another rule to the file:
##### 4) Allow routing of all **ICMP** protocol packets
![7.1.8](../misc/images/report_img/7.1.8.png)
##### Run the file as in Part 4
![7.1.9](../misc/images/report_img/7.1.9.png)
##### Check connection between ws22 and r1 with the `ping` command
*When running the file with these rules, ws22 should ping from r1*

![7.1.10](../misc/images/report_img/7.1.10.png)
##### Add two more rules to the file:
##### 5) Enable **SNAT**, which is masquerade all local ip from the local network behind r2 (as defined in Part 5 - network 10.20.0.0)
*Tip: it is worth thinking about routing internal packets as well as external packets with an established connection*
##### 6) Enable **DNAT** on port 8080 of r2 machine and add external network access to the Apache web server running on ws22
*Tip: be aware that when you will try to connect, there will be a new tcp connection for ws22 and port 80

![7.1.14](../misc/images/report_img/7.1.14.png)
##### Run the file as in Part 4
![7.1.13](../misc/images/report_img/7.1.13.png)

*Before testing it is recommended to disable the **NAT** network interface in VirtualBox (its presence can be checked with `ip a` command), if it is enabled*
##### Check the TCP connection for **SNAT** by connecting from ws22 to the Apache server on r1 with the `telnet 10.10.0.1 80` command
![7.1.15](../misc/images/report_img/7.1.15.png)
##### Check the TCP connection for **DNAT** by connecting from r1 to the Apache server on ws22 with the `telnet 10.100.0.12 8080` 
![7.1.16](../misc/images/report_img/7.1.16.png)

##### Save dumps of virtual machine images

## Part 8. Bonus. Introduction to **SSH Tunnels**

##### Run a firewall on r2 with the rules from Part 7
![7.1.13](../misc/images/report_img/7.1.13.png)
##### Start the **Apapche** web server on ws22 on localhost
![8.1](../misc/images/report_img/8.1.png)
##### only (i.e. in */etc/apache2/ports.conf* file change the line `Listen 80` to `Listen localhost:80`)
![8.0](../misc/images/report_img/8.0.png)
##### Use *Local TCP forwarding* from ws21 to ws22 to access the web server on ws22 from ws21
![8.4](../misc/images/report_img/8.4.png)
##### Use *Remote TCP forwarding* from ws11 to ws22 to access the web server on ws22 from ws11
![8.4](../misc/images/report_img/8.2.png)
##### To check if the connection worked in both of the previous steps, go to a second terminal (e.g. with the Alt + F2) and run the `telnet 127.0.0.1 [local port]` command.
##### ws11:
![8.3](../misc/images/report_img/8.3.png)
##### ws21:
![8.5](../misc/images/report_img/8.5.png)

##### Save dumps of virtual machine images
