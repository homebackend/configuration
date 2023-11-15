You have a ssh machine ServerA with IP address `192.168.1.101` in city A. You connect to it using your laptop's ssh command like so `ssh ServerA`.
Further you have a ssh machine ServerB with IP address `192.168.1.102` in city B. In city B you have reserved IP address `192.168.1.101` and not assigned it to any machine.
The two servers belong to different subnets (each in their own city), even though IP address assigned are
As you move from city A to city B you want to access ServerA using ssh with the same command as before, viz. `ssh ServerA`.

To achieve this first we need to tunnel ssh to 
1. Assign ServerB and additional IP address `192.168.1.101` as documented [here](add-additional-static-ips.md). This ensures that in city B now you have a machine that responds to network requests sent to IP address `192.168.1.101`.
2. Forward port 22 from ServerA to port 2222 ServerA by creating ssh tunnel as documented [here](ssh-tunnel-over-internet.md).

At this point, you can connect to port 2222 on IP address `192.168.1.101` to connect to ServerA using the command `ssh -p 2222 ServerA`, even though there is no ServerA in city B.
This is good, but not good enough to transparently connect to ServerA using the command `ssh ServerA`. When we execute the command `ssh ServerA` the request will end up at port 22 on ServerB, since even though we have assigned ServerB an IP same as ServerA, on port 22 ssh runnig on ServerB will respond.
So what we need to do is to forward requests coming to port 22 to port 2222, if and only if, request is coming on IP addresss `192.168.1.101`. For IP address `192.168.1.102` there is no forwarding. This is what we intend to achieve here.

# Forward port to another IP address using iptables

The folowing two commands will enable destination network address translation (DNAT) where source IP address is `192.168.1.101` for destination port (dport) 22 to destination IP address `192.168.1.102` and port `2222`.

```sh
iptables -A PREROUTING -d 192.168.1.101/32 -p tcp -m tcp --dport 22 -j DNAT --to-destination 192.168.1.102:2222
iptables -A OUTPUT -d 192.168.1.101/32 -p tcp -m tcp --dport 22 -j DNAT --to-destination 192.168.1.102:2222
```

Note, OUTPUT caters to network requests originating from within ServerB, while PREROUTING caters to network requests originating from outside of ServerB (i.e. some other machine).

The final configuration can be represented as follows:

![Diagram](http://cdn-0.plantuml.com/plantuml/png/bL9DJ-im4BpxLwnoUPUNlXUtHO1GrPOB1mv8I5mhasoB9TUsDfiYGFtlk9j0IFcGE7XRxOndrfETrso8cwrJ3jSC783ive6XieGbs-3L7xWAP9-3P-F0MO_rEUJvx2zSvBcKMQBS8R4jFn04JoUXRDaXt0HYM0TwD5HPlm4cu-je2Bsueg-WgP6KMWTN8K5sQVa9bXcyA80oR6Fm1sfsRIA7-AyuqwFYv7HKZFJ_8WDJWnD0w00n48ScqLsBhGezdamJj4zd87YBj4DQohs1KUzGT0wxRSlgzAs7J6j1ucwbDGmnwMajpRtKdnnQ7Vha42cf7BS6mkdibIfTQ2gmdf4yyb6AYz-cUFzTZDF9M1VY-KM82xdLp-baz7Q-xWiSDgTDHacL_Nb_aVnspVujRt-atQaEzIIZRZZeINFqLThiLm00)

