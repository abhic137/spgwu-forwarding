# spgwu-forwarding
in core pc
```
sudo bash createLink.sh oai-spgwu ubuntu
C1_IF=$(sudo ip netns exec oai-spgwu ifconfig -a | grep -E "dcp.*"| awk -F':' '{print $1}')
sudo ip netns exec oai-spgwu ip a add 10.0.0.1/30 dev ${C1_IF}
sudo ip netns exec oai-spgwu ip r add 10.0.0.2/32 via 0.0.0.0 dev ${C1_IF}
sudo ip netns exec ubuntu ip a add 10.0.0.2/30 dev dcp1

```
in spgwu
```
ip route del default via 192.168.70.129 dev eth0 
sysctl net.ipv4.ip_forward=1
iptables -P FORWARD ACCEPT
ip route add default via 10.0.0.2 dev dcp1
```
spgwu routing table
```
ip route
default via 10.0.0.2 dev dcp1 
10.0.0.0/30 dev dcp1 proto kernel scope link src 10.0.0.1 
10.0.0.2 dev dcp1 
12.1.1.0/24 dev tun0 proto kernel scope link src 12.1.1.1 
192.168.70.128/26 dev eth0 proto kernel scope link src 192.168.70.134 
```
## Observation
packet captures at ubuntu's dcp1 interface show the UE ip 12.1.1.51 is able to hit the ubuntu machine's dcp1 interface.
```
#in UE ping 8.8.8.8
#in ubuntu service
tcpdump -i dcp1
#results
 
08:53:09.799446 IP 12.1.1.51 > dns.google: ICMP echo request, id 7, seq 1, length 64
08:53:09.969429 IP 12.1.1.51.65261 > 172.21.3.100.53: 16960+ A? beacons3.gvt2.com. (35)
08:53:09.970055 IP 12.1.1.51.3305 > 172.21.3.100.53: 27436+ A? beacons3.gvt2.com. (35)
08:53:10.779515 IP 12.1.1.51 > dns.google: ICMP echo request, id 7, seq 2, length 64
08:53:11.089380 IP 12.1.1.51.10805 > dns.google.53: 14842+ A? beacons5.gvt2.com. (35)
08:53:11.799507 IP 12.1.1.51 > dns.google: ICMP echo request, id 7, seq 3, length 64
08:53:12.819438 IP 12.1.1.51.52591 > dns.google.53: 7503+ A? optimizationguide-pa.googleapis.com. (53)
08:53:12.829208 IP 12.1.1.51 > dns.google: ICMP echo request, id 7, seq 4, length 64
08:53:13.849419 IP 12.1.1.51 > dns.google: ICMP echo request, id 7, seq 5, length 64
08:53:14.869467 IP 12.1.1.51 > dns.google: ICMP echo request, id 7, seq 6, length 64
08:53:14.979375 IP 12.1.1.51.57876 > dns.google.53: 27436+ A? beacons3.gvt2.com. (35)
08:53:14.979842 IP 12.1.1.51.43858 > dns.google.53: 16960+ A? beacons3.gvt2.com. (35)


```
## adding route to ubuntu
```
##ubuntu should be connected to the oai network 
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
ip route add 12.1.1.0/24 via 192.168.70.134 dev eth0

##OUTPUT
#ip route sh
#default via 192.168.70.129 dev eth0 
#10.0.0.0/30 dev dcp2 proto kernel scope link src 10.0.0.2 
#12.1.1.0/24 via 192.168.70.134 dev eth0 
#172.20.0.0/16 dev eth1 proto kernel scope link src 172.20.0.3 
#192.168.70.128/26 dev eth0 proto kernel scope link src 192.168.70.135 

```
##OBSERVATION
```

root@ubuntu:/# tcpdump -i eth0
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
11:11:52.015880 IP ubuntu > dns.google: ICMP echo request, id 19, seq 1, length 64
11:11:52.032068 IP dns.google > ubuntu: ICMP echo reply, id 19, seq 1, length 64
11:11:52.032081 IP dns.google > 12.1.1.51: ICMP echo reply, id 19, seq 1, length 64
11:11:52.032134 IP ubuntu.2152 > 192.168.71.194.2152: UDP, length 100
11:11:52.995639 IP ubuntu > dns.google: ICMP echo request, id 19, seq 2, length 64
11:11:53.011740 IP dns.google > ubuntu: ICMP echo reply, id 19, seq 2, length 64
11:11:53.011751 IP dns.google > 12.1.1.51: ICMP echo reply, id 19, seq 2, length 64
11:11:53.011789 IP ubuntu.2152 > 192.168.71.194.2152: UDP, length 100
11:11:53.995779 IP ubuntu > dns.google: ICMP echo request, id 19, seq 3, length 64
11:11:54.011738 IP dns.google > ubuntu: ICMP echo reply, id 19, seq 3, length 64
11:11:54.011749 IP dns.google > 12.1.1.51: ICMP echo reply, id 19, seq 3, length 64
11:11:54.011788 IP ubuntu.2152 > 192.168.71.194.2152: UDP, length 100

```
