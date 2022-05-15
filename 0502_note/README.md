### ssh tunnel 
### ex1. 
![20220502-1](/img/20220502-1.jpg)
- lab4.py
```py
#!/usr/bin/python
from mininet.net import Containernet
from mininet.node import Docker
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.link import TCLink, Link
 
def topology():
 
    "Create a network with some docker containers acting as hosts."
    net = Containernet()
 
    info('*** Adding hosts\n')
    h1 = net.addHost('h1', ip='192.168.0.1/24')
    h2 = net.addHost('h2', ip='192.168.0.2/24')
    br1 = net.addHost('br1')
    r1 = net.addHost('r1', ip='192.168.0.254/24')
    d1 = net.addDocker('d1', ip='10.0.0.1/24', dimage="ubuntu:2.0")
 
    info('*** Creating links\n')
    net.addLink(h1, br1)
    net.addLink(h2, br1)
    net.addLink(r1, br1)
    net.addLink(r1, d1)
   
    info('*** Starting network\n')
    net.start()
    d1.cmd("/etc/init.d/ssh start")
    r1.cmd("ifconfig r1-eth1 0")
    r1.cmd("ip addr add 10.0.0.2/24 brd + dev r1-eth1")
    r1.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r1.cmd("iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o r1-eth1 -j MASQUERADE")
    h1.cmd("ip route add default via 192.168.0.254")
    br1.cmd("ifconfig br1-eth0 0")
    br1.cmd("ifconfig br1-eth1 0")
    br1.cmd("ifconfig br1-eth2 0")
    br1.cmd("brctl addbr br1")
    br1.cmd("brctl addif br1 br1-eth0")
    br1.cmd("brctl addif br1 br1-eth1")
    br1.cmd("brctl addif br1 br1-eth2")
    br1.cmd("ifconfig br1 up") 
 
    info('*** Running CLI\n')
    CLI(net)
 
    info('*** Stopping network')
    net.stop()
 
if __name__ == '__main__':
    setLogLevel('info')
    topology()

```
### 執行 lab4.py
### containernet> xterm h1 h2 
### 另一個終端機 docker exec -it mn.d1 bash
### h2> python -m SimpleHTTPServer 80
### h1> ssh -Nf -R 10.0.0.1:5555:192.168.0.2:80 root@10.0.0.1
### d1> curl 127.0.0.1:5555
### ex2.
![20220502-2](/img/20220502-2.jpg)
- lab5.py
```py
#!/usr/bin/python
from mininet.net import Containernet
from mininet.node import Docker
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.link import TCLink, Link
 
def topology():
 
    "Create a network with some docker containers acting as hosts."
    net = Containernet()
 
    info('*** Adding hosts\n')
    h1 = net.addHost('h1', ip='192.168.0.1/24')
    r1 = net.addHost('r1', ip='192.168.0.254/24')
    d1 = net.addDocker('d1', ip='10.0.0.1/24', dimage="ubuntu:2.0")
    br1 = net.addHost('br1')
    h2 = net.addHost('h2', ip='10.0.0.3/24')
    h3 = net.addHost('h3', ip='10.0.0.4/24')
 
    info('*** Creating links\n')
    net.addLink(h1, r1)
    net.addLink(r1, br1)
    net.addLink(d1, br1)
    net.addLink(h2, br1)
    net.addLink(h3, br1)
   
    info('*** Starting network\n')
    net.start()
    d1.cmd("/etc/init.d/ssh start")
    r1.cmd("ifconfig r1-eth1 0")
    r1.cmd("ip addr add 10.0.0.2/24 brd + dev r1-eth1")
    r1.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r1.cmd("iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o r1-eth1 -j MASQUERADE")
    r1.cmd("iptables -A FORWARD -s 192.168.0.1 -p tcp --dport 80 -j REJECT")
    h1.cmd("ip route add default via 192.168.0.254")
    br1.cmd("ifconfig br1-eth0 0")
    br1.cmd("ifconfig br1-eth1 0")
    br1.cmd("ifconfig br1-eth2 0")
    br1.cmd("ifconfig br1-eth3 0")
    br1.cmd("brctl addbr br1")
    br1.cmd("brctl addif br1 br1-eth0")
    br1.cmd("brctl addif br1 br1-eth1")
    br1.cmd("brctl addif br1 br1-eth2")
    br1.cmd("brctl addif br1 br1-eth3")
    br1.cmd("ifconfig br1 up") 
   
    info('*** Running CLI\n')
    CLI(net)
 
    info('*** Stopping network')
    net.stop()
 
if __name__ == '__main__':
    setLogLevel('info')
    topology()
```
### 執行 lab5.py
### containernet> xterm h1 h2 h3
### h2> python -m SimpleHTTPServer 80
### h3> python -m SimpleHTTPServer 8080
### h1> ssh -Nf -D 127.0.0.1:8080 root@10.0.0.1
### h1> su - user
### h1> firefox
### Refrence下的Setwork Settings
![20220502-3](/img/20220502-3.jpg)
![20220502-4](/img/20220502-4.jpg)
## 軟體定義網路 software-defined networking SDN
它利用OpenFlow協定將路由器的控制平面（control plane）從資料平面（data plane）中分離，改以軟體方式實作，從而使得將分散在各個網路裝置上的控制平面進行集中化管理成為可能 ，該架構可使網路管理員在不更動硬體裝置的前提下，以中央控制方式用程式重新規劃網路，為控制網路流量提供了新方案，也為核心網路和應用創新提供了良好平台。
![20220502-5](/img/20220502-5.jpg)
### ex1.
### mn --topo single,2
![20220502-6](/img/20220502-6.jpg)
### mininet-wifi> sh ovs-ofctl dump-flows s1 ### 查看規則
### 新增規則 :
### mininet-wifi> sh ovs-ofctl add-flow s1 in_port=1,actions=output:2 ### 1號口進,2號口出
### mininet-wifi> sh ovs-ofctl add-flow s1 in_port=2,actions=output:1 ### 2號口進,1號口出
### 刪除規則 :
### mininet-wifi> sh ovs-ofctl del-flows s1