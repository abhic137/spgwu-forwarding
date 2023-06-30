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
