This readme document's configuring a network interface on GNU/Linux with multiple ip addresses.

To start with, you have a configuration like:

```sh
# ip -4 addr list dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    inet 192.168.1.66/24 brd 192.168.1.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
```
Here the interface `eth0` has been assigned an IP Addres `192.168.1.66` using **dhcp**.

To assign another IP address to the same interface create a file `99-intranet-ip-assign` in the directory `/usr/lib/dhcpcd/dhcpcd-hooks` with the following contents:

```sh
ip addr add 192.168.1.64/32 dev eth0:1
ip addr add 192.168.1.65/32 dev eth0:2
```
and restart the machine. Feel free to rename the file as per your liking.

On reboot, you will see a configuration like this:

```sh
# ip -4 addr list dev eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    inet 192.168.1.64/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.1.65/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.1.66/24 brd 192.168.1.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
```
As you can see two new IP addresses have been assigned to interface `eth0`. At this point you can try pinging these interfaces from another machine to confirm that they are working as desired.
