## 使用 python 腳本編寫情境描述檔
### 練習1
- ![20220307-1](/img/20220307-1.jpg)
### 編寫 1.py
```py
#!/usr/bin/python

from mininet.cli import CLI  #command line interface
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  Link(h1,h2)
  net.build()
  CLI(net)  #命令提示符號
  net.stop()
```
### 執行 1.py
- chomod +x 1.py
- ./1.py
- mininet> xterm h1 h2
### h1 h2 互 ping 可通
### 配置 ip
- h1> ifconfig h1-eth0 0
- h1> ip addr add 192.168.1.1/24 brd + dev h1-eth0
- h2> ifconfig h2-eth0 0
- h2> ip a a 192.168.1.1/24 brd + dev h1-eth0
### 練習2
### 編寫 2.py
```py
#!/usr/bin/python

from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
net=Mininet(link=TCLink)
h1=net.addHost('h1')
h2=net.addHost('h2')
Link(h1,h2)
net.build()
h1.cmd("ifconfig h1-eth0 0")
h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")
h2.cmd("ifconfig h2-eth0 0")
h2.cmd("ip a a 192.168.1.2/24 brd + dev h2-eth0")
CLI(net)
net.stop()
```
### 練習3 
- ![20220307-2](/img/20220307-2.jpg)
### 編寫 3.py
```py
#!/usr/bin/python

from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  h3=net.addHost('h3')
  Link(h1,h2)
  Link(h2,h3)
  net.build()
  h1.cmd("ifconfig h1-eth0 0")
  h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")

  h2.cmd("ifconfig h2-eth0 0")
  h2.cmd("ip a a 192.168.1.2/24 brd + dev h2-eth0")
  h2.cmd("ifconfig h2-eth1 0")
  h2.cmd("ip a a 192.168.2.2/24 brd + dev h2-eth1")

  h3.cmd("ifconfig h3-eth0 0")
  h3.cmd("ip a a 192.168.2.1/24 brd + dev h3-eth0")

  h1.cmd("ip route add default via 192.168.1.2") # 加入內定路由
  h3.cmd("ip route add default via 192.168.2.2")
  h2.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")

  CLI(net)
  net.stop()
``` 
### 練習4
- ![20220307-3](/img/20220307-3.jpg)
- ![20220307-4](/img/20220307-4.jpg)
### 編寫 4.py
```py
#!/usr/bin/python

from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  h3=net.addHost('h3')
  br0=net.addHost('br0')
  Link(h1,br0)
  Link(h2,br0)
  Link(h3,br0)
  net.build()
  br0.cmd("brctl addbr mybr")
  br0.cmd("brctl addif mybr br0-eth0")
  br0.cmd("brctl addif mybr br0-eth1")
  br0.cmd("brctl addif mybr br0-eth2")
  br0.cmd("ifconfig mybr up")
  CLI(net)
  net.stop()
```
- 更改網路卡卡號
```
  h1.cmd("ifconfig h1-eth0 down")
  h1.cmd("ifconfig h1-eth0 hw ether 00:00:00:00:00:01")
  h1.cmd("ifconfig h1-eth0 up")
```
## arp poisoning
### 安裝 dsniff
- apt install dsniff
### 情境
- ![20220307-5](/img/20220307-5.jpg)
### h3 攻擊 h1 h2
- h3> arpspoof -i h3-eth0 -t 10.0.0.1 10.0.0.2
- h3> arpspoof -i h3-eth0 -t 10.0.0.2 10.0.0.1
### 攻擊後 h1 h2 arp -n 
- ![20220307-6](/img/20220307-6.jpg)
### 解決方法-使用靜態arp
- h1> arp -s 10.0.0.2 00:00:00:00:00:02
- h2> arp -s 10.0.0.1 00:00:00:00:00:01
