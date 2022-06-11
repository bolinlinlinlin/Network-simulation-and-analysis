## 期中考
### ![20220418-1](/img/20220418-1.jpg)
- 1.py
```py
#!/usr/bin/python
from mininet.net import Mininet
from mininet.link import Link, TCLink
from mininet.cli import CLI
from mininet.log import setLogLevel
 
def topology():
    "Create a network."
    net = Mininet()
 
    
    h1 = net.addHost( 'h1', ip="1.1.1.1/24")
    h2 = net.addHost( 'h2', ip="2.2.2.2/24")
    r1 = net.addHost( 'r1')
    r2 = net.addHost( 'r2')
    r3 = net.addHost( 'r3')
 
 
    
    net.addLink(h1, r1)
    net.addLink(r1, r3)
    net.addLink(r3, h2)
    net.addLink(r2, r3)
    net.addLink(r2, r1)
    
    
    net.build()
 
    
    r1.cmd("ifconfig r1-eth0 0")
    r1.cmd("ifconfig r1-eth1 0")
    r1.cmd("ifconfig r1-eth2 0")
    r2.cmd("ifconfig r2-eth0 0")
    r2.cmd("ifconfig r2-eth1 0")
    r2.cmd("ifconfig r2-eth2 0")
    r3.cmd("ifconfig r3-eth0 0")
    r3.cmd("ifconfig r3-eth1 0")
    r3.cmd("ifconfig r3-eth2 0")
    h1.cmd("ifconfig h1-eth0 0")
    h2.cmd("ifconfig h2-eth0 0")

    h1.cmd("ip a a 1.1.1.1/24 brd + dev h1-eth0")
    h2.cmd("ip a a 2.2.2.2/24 brd + dev h2-eth0")

    r1.cmd("ip a a 1.1.1.254/24 brd + dev r1-eth0")
    r1.cmd("ip a a 192.168.1.1/24 brd + dev r1-eth1")
    r1.cmd("ip a a 192.168.2.1/24 brd + dev r1-eth2")
    
    r2.cmd("ip a a 192.168.3.2/24 brd + dev r2-eth0")
    r2.cmd("ip a a 192.168.2.2/24 brd + dev r2-eth1")
    
    r3.cmd("ip a a 192.168.1.2/24 brd + dev r3-eth0")
    r3.cmd("ip a a 2.2.2.254/24 brd + dev r3-eth1")
    r3.cmd("ip a a 192.168.3.1/24 brd + dev r3-eth2")
    
    h1.cmd("ip route add default via 1.1.1.254")
    h2.cmd("ip route add default via 2.2.2.254")
    
    ####  h1(0) --- (0)r1(1) --- (0)r3(1) ---(0)h2
    ####              (2)          (2)
    ####                \          /
    ####                  (1)r2(0)
    
    r1.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r2.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r3.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r1.cmd("ip route add 192.168.2.0 via 192.168.1.2")
    r3.cmd("ip route add 192.168.1.0 via 192.168.3.2")
    r2.cmd("ip route add 192.168.1.0 via 192.168.2.1")
    CLI( net )
 
   
    net.stop()
 
if __name__ == '__main__':
    setLogLevel( 'info' )
    topology()

```
### ![20220418-2](/img/20220418-2.jpg)
- 2.py
```py
#!/usr/bin/env python
from mininet.net import Mininet
from mininet.cli import CLI
from mininet.link import Link,TCLink,Intf

if '__main__' == __name__:
  net = Mininet(link=TCLink)
  #h1 is under vlan10
  h1 = net.addHost('h1')
  #h2 is under vlan20
  h2 = net.addHost('h2')
  #h3 is under vlan10
  h3 = net.addHost('h3')
  #h4 is under vlan20
  h4 = net.addHost('h4')
  #s1 is a switch
  s1 = net.addHost('s1')
  #s2 is a switch
  s2 = net.addHost('s2')
  
  Link(h1, s1)
  Link(h2, s1)
  Link(h3, s2)
  Link(h4, s2)
  Link(s1, s2)
  net.build()
  
  s1.cmd("ifconfig s1-eth0 0")
  s1.cmd("ifconfig s1-eth1 0")
  s1.cmd("ifconfig s1-eth2 0")
  s2.cmd("ifconfig s2-eth0 0")
  s2.cmd("ifconfig s2-eth1 0")
  s2.cmd("ifconfig s2-eth2 0")
  s1.cmd("vconfig add s1-eth2 10")
  s1.cmd("vconfig add s1-eth2 20")
  s2.cmd("vconfig add s2-eth2 10")
  s2.cmd("vconfig add s2-eth2 20")
  s1.cmd("ifconfig s1-eth2.10 up")
  s1.cmd("ifconfig s1-eth2.20 up")
  s2.cmd("ifconfig s2-eth2.10 up")
  s2.cmd("ifconfig s2-eth2.20 up")
  s1.cmd("brctl addbr brvlan10")
  s1.cmd("brctl addbr brvlan20")
  s1.cmd("brctl addif brvlan10 s1-eth0")
  s1.cmd("brctl addif brvlan20 s1-eth1")
  s1.cmd("brctl addif brvlan10 s1-eth2.10")
  s1.cmd("brctl addif brvlan20 s1-eth2.20")
  s2.cmd("brctl addbr brvlan10")
  s2.cmd("brctl addbr brvlan20")
  s2.cmd("brctl addif brvlan10 s2-eth0")
  s2.cmd("brctl addif brvlan20 s2-eth1")
  s2.cmd("brctl addif brvlan10 s2-eth2.10")
  s2.cmd("brctl addif brvlan20 s2-eth2.20")
  s1.cmd("ifconfig brvlan10 up")
  s1.cmd("ifconfig brvlan20 up")
  s2.cmd("ifconfig brvlan10 up")
  s2.cmd("ifconfig brvlan20 up")
  h1.cmd("ifconfig h1-eth0 10.0.10.1 netmask 255.255.255.0")
  h2.cmd("ifconfig h2-eth0 10.0.10.2 netmask 255.255.255.0")
  h3.cmd("ifconfig h3-eth0 10.0.10.3 netmask 255.255.255.0")
  h4.cmd("ifconfig h4-eth0 10.0.10.4 netmask 255.255.255.0")
  CLI(net)
  net.stop()

```
### ![20220418-3](/img/20220418-3.jpg)
- 3.py
```py 
#!/usr/bin/env python
from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__' == __name__:
  net = Mininet(link=TCLink)
  h1 = net.addHost('h1')
  h2 = net.addHost('h2')
  h3 = net.addHost('h3')
  h4 = net.addHost('h4')
  h5 = net.addHost('h5')
  h6 = net.addHost('h6')
  r1 = net.addHost('r1')
  r2 = net.addHost('r2')
  h1r1 = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h1, r1, cls=TCLink , **h1r1)
  h2r1 = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h2, r1, cls=TCLink , **h2r1)
  h3r1 = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h3, r1, cls=TCLink , **h3r1)
  h4r2 = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h4, r2, cls=TCLink , **h4r2)
  h5r2 = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h5, r2, cls=TCLink , **h5r2)
  h6r2 = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h6, r2, cls=TCLink , **h6r2)
  r1r2 = {'bw':10,'delay':'1ms','loss':0}
  net.addLink(r1, r2, cls=TCLink , **r1r2)
  
  
  net.build()
  h1.cmd("ifconfig h1-eth0 0")
  h2.cmd("ifconfig h2-eth0 0")
  h3.cmd("ifconfig h3-eth0 0")
  h4.cmd("ifconfig h4-eth0 0")
  h5.cmd("ifconfig h5-eth0 0")
  h6.cmd("ifconfig h6-eth0 0")
  r1.cmd("ifconfig r1-eth0 0")
  r1.cmd("ifconfig r1-eth1 0")
  r1.cmd("ifconfig r1-eth2 0")
  r1.cmd("ifconfig r1-eth3 0")
  r2.cmd("ifconfig r2-eth0 0")
  r2.cmd("ifconfig r2-eth1 0")
  r2.cmd("ifconfig r2-eth2 0")
  r2.cmd("ifconfig r2-eth3 0")
  r1.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
  r2.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
  h1.cmd("ip addr add 1.1.1.1/24 brd + dev h1-eth0")
  h2.cmd("ip addr add 2.2.2.2/24 brd + dev h2-eth0")
  h3.cmd("ip addr add 3.3.3.3/24 brd + dev h3-eth0")
  h4.cmd("ip addr add 4.4.4.4/24 brd + dev h4-eth0")
  h5.cmd("ip addr add 5.5.5.5/24 brd + dev h5-eth0")
  h6.cmd("ip addr add 6.6.6.6/24 brd + dev h6-eth0")
  r1.cmd("ip addr add 1.1.1.254/24 brd + dev r1-eth0")
  r1.cmd("ip addr add 2.2.2.254/24 brd + dev r1-eth1")
  r1.cmd("ip addr add 3.3.3.254/24 brd + dev r1-eth2")
  r1.cmd("ip addr add 10.0.0.1/24 brd + dev r1-eth3")
  r2.cmd("ip addr add 4.4.4.254/24 brd + dev r2-eth0")
  r2.cmd("ip addr add 5.5.5.254/24 brd + dev r2-eth1")
  r2.cmd("ip addr add 6.6.6.254/24 brd + dev r2-eth2")
  r2.cmd("ip addr add 10.0.0.2/24 brd + dev r2-eth3")
  h1.cmd("ip route add default via 1.1.1.254")
  h2.cmd("ip route add default via 2.2.2.254")
  h3.cmd("ip route add default via 3.3.3.254")
  h4.cmd("ip route add default via 4.4.4.254")
  h5.cmd("ip route add default via 5.5.5.254")
  h6.cmd("ip route add default via 6.6.6.254")
  r1.cmd("ip route add 4.4.4.0/24 via 10.0.0.2")
  r1.cmd("ip route add 5.5.5.0/24 via 10.0.0.2")
  r1.cmd("ip route add 6.6.6.0/24 via 10.0.0.2")
  r2.cmd("ip route add 1.1.1.0/24 via 10.0.0.1")
  r2.cmd("ip route add 2.2.2.0/24 via 10.0.0.1")
  r2.cmd("ip route add 3.3.3.0/24 via 10.0.0.1")
  CLI(net)
  net.stop()
```
### ![20220418-4](/img/20220418-4.jpg)
- 4.py
```py
#!/usr/bin/python
from mininet.net import Mininet
from mininet.link import Link, TCLink
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.net import Containernet
from mininet.node import Controller, Docker, OVSSwitch

 
def topology():
    "Create a network."
    net = Containernet()
 
    #print "*** Creating nodes"
    h1 = net.addHost( 'h1', ip='192.168.1.1/24')
    r1 = net.addHost( 'r1')

    info('*** Adding docker containers\n')
    d1 = net.addDocker('d1', ip='1.1.1.1', dimage="ubuntu:1.0")
    d2 = net.addDocker('d2', ip='2.2.2.2', dimage="ubuntu:1.0")
    #print "*** Creating links"
    net.addLink(h1, r1)
    net.addLink(r1, d1)
    net.addLink(r1, d2)
 
    #print "*** Starting network"
    net.start()
 
    #print "*** Running CLI"
    r1.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r1.cmd("ifconfig r1-eth0 0")
    r1.cmd("ifconfig r1-eth1 0")
    r1.cmd("ifconfig r1-eth2 0")
    r1.cmd("ip addr add 192.168.1.254/24 brd + dev r1-eth0")
    r1.cmd("ip addr add 1.1.1.254/24 brd + dev r1-eth1")
    r1.cmd("ip addr add 2.2.2.254/24 brd + dev r1-eth2")

    h1.cmd("ip route add default via 192.168.1.254")
    d1.cmd("ip route add default via 1.1.1.254")
    d2.cmd("ip route add default via 2.2.2.254")

    r1.cmd("iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o r1-eth1 -j MASQUERADE")
    r1.cmd("iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o r1-eth2 -j MASQUERADE")

    d1.cmd("/etc/init.d/apache2 start")
    d2.cmd("/etc/init.d/ssh start")
 
    CLI(net)
 
    #print "*** Stopping network"exit

    net.stop()
 
if __name__ == '__main__':
    setLogLevel( 'info' )
    topology()
```