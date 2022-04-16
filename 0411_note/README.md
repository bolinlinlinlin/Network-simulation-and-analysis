### 確認能否跑 Dockernet
```
cd /home/user/containernet
python3 ./setup.py install
cd /home/user/containernet/example
python3 dockerhost.py


cd /home/user/mininet-wifi
util/install.sh -n


/etc/init.d/ssh stop
mn --topo single,3
xterm h1 h2 h3
in h2-->/usr/sbin/sshd && netstat -tunlp | grep sshd
in h3-->/usr/sbin/sshd && netstat -tunlp | grep sshd
in h1-->ssh user@10.0.0.2  or ssh user@10.0.0.3

```
### 查看有無 ubuntu:trusty 的鏡像 
- docker images
![20220411-1](/img/20220411-1.jpg)
### 如果沒有, 使用 docker pull ubuntu:trusty

### dockerhost1.py 
```py
#!/usr/bin/python

"""
This example shows how to create a simple network and
how to create docker containers (based on existing images)
to it.
"""

from mininet.net import Containernet
from mininet.node import Controller, Docker, OVSSwitch
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.link import TCLink, Link


def topology():

    "Create a network with some docker containers acting as hosts."

    net = Containernet()

    info('*** Adding hosts\n')
    h1 = net.addHost('h1',  ip='10.0.0.250')

    info('*** Adding docker containers\n')
    d1 = net.addDocker('d1', ip='10.0.0.251', dimage="ubuntu:trusty")
    
    info('*** Creating links\n')
    net.addLink(h1, d1)
   
    info('*** Starting network\n')
    net.start()

    info('*** Running CLI\n')
    CLI(net)

    info('*** Stopping network')
    net.stop()

if __name__ == '__main__':
    setLogLevel('info')
    topology()

```
### 執行 dockerhost1.py 
![20220411-2](/img/20220411-2.jpg)
- xterm h1
### docker 的終端機另開一個視窗
### 查看 docker 名稱
- docker ps
### 進入 docker 環境
- docker exec -it mn.dl bash
-----
### 在 dockernet 環境下安裝軟體
- docker run -it ubuntu:trusty bash
- apt update 
### 安裝 openssh-server 
- apt install openssh-server 
### 安裝網頁伺服器 
- apt install apache2
### 安裝 vim    
- apt install vim
### 修改 ssh 設定 ( 預設ssh server是不允許root遠端登入 )
- vim /etc/ssh/sshd_config
### 按下 "/" 
- /PermitRoot 
- PermitRoot without-password 改為 PermitRootLogin yes
### 啟動 ssh
- /etc/init.d/ssh start
- /etc/init.d/ssh status
### 設定 ssh root password 
- passwd root 
### 啟動網頁伺服器
- /etc/init.d/apache2 start
- /etc/init.d/apache2/status
### 把現有狀態製作成新鏡像
- docker commit (docker的名字) ubuntu:(新名字)![20220411-3](/img/20220411-3.jpg)
### 修改 dockerhost1.py 
```py
#!/usr/bin/python

"""
This example shows how to create a simple network and
how to create docker containers (based on existing images)
to it.
"""

from mininet.net import Containernet
from mininet.node import Controller, Docker, OVSSwitch
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.link import TCLink, Link


def topology():

    "Create a network with some docker containers acting as hosts."

    net = Containernet()

    info('*** Adding hosts\n')
    h1 = net.addHost('h1',  ip='10.0.0.250')

    info('*** Adding docker containers\n')
    d1 = net.addDocker('d1', ip='10.0.0.251', dimage="ubuntu:1.0")
    
    info('*** Creating links\n')
    net.addLink(h1, d1)
   
    info('*** Starting network\n')
    net.start()
    d1.cmd("/etc/init.d/ssh start")
    d1.cmd("/etc/init.d/apache2 start")

    info('*** Running CLI\n')
    CLI(net)

    info('*** Stopping network')
    net.stop()

if __name__ == '__main__':
    setLogLevel('info')
    topology()
```
### 執行 dockerhost1.py
- python3 dockerhost1.py
### h1 ssh root@10.0.0.251
![20220411-4](/img/20220411-4.jpg)

----

### 環境
![20220411-5](/img/20220411-5.jpg)
### 架構
![20220411-6](/img/20220411-6.jpg)

### 如何連進私有網路
- 1.port forwarding (DNAT)
- 2.frp 
### frp server 與  frp client  連接 ， 外部網路就可以透過 frp server 來存取內部網路
### cd /home/user/server-test/test-frp
### test.py
```py
#!/usr/bin/python
from mininet.net import Mininet
from mininet.link import Link, TCLink
from mininet.cli import CLI
from mininet.log import setLogLevel
 
def topology():
    "Create a network."
    net = Mininet()
 
    print "*** Creating nodes"
    h1 = net.addHost( 'h1', ip="192.168.1.1/24") #private server
    h2 = net.addHost( 'h2', ip="1.1.1.1/24") #public server
    h3 = net.addHost( 'h3', ip="2.2.2.2/24") #public node
    r1 = net.addHost( 'r1')
    r2 = net.addHost( 'r2')
 
    ####  h1 --- r1 ---r2----h3
    ####               |
    ####               h2
 
    print "*** Creating links"
    net.addLink(h1, r1)
    net.addLink(r1, r2)
    net.addLink(r2, h2)
    net.addLink(r2, h3)
 
    print "*** Starting network"
    net.build()
 
    print "*** Running CLI"
    r1.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r2.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
    r1.cmd("ifconfig r1-eth0 0")
    r1.cmd("ifconfig r1-eth1 0")
    r2.cmd("ifconfig r2-eth0 0")
    r2.cmd("ifconfig r2-eth1 0")
    r2.cmd("ifconfig r2-eth2 0")
    r1.cmd("ip addr add 192.168.1.254/24 brd + dev r1-eth0")
    r1.cmd("ip addr add 12.1.1.1/24 brd + dev r1-eth1")
    r2.cmd("ip addr add 12.1.1.2/24 brd + dev r2-eth0")
    r2.cmd("ip addr add 1.1.1.254/24 brd + dev r2-eth1")
    r2.cmd("ip addr add 2.2.2.254/24 brd + dev r2-eth2")
    h1.cmd("ip route add default via 192.168.1.254")
    h2.cmd("ip route add default via 1.1.1.254")
    h3.cmd("ip route add default via 2.2.2.254")
    r2.cmd("ip route add 12.1.1.0/24 via 12.1.1.1")
    r1.cmd("ip route add 1.1.1.0/24 via 12.1.1.2")
    r1.cmd("ip route add 2.2.2.0/24 via 12.1.1.2")
    r1.cmd("iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o r1-eth1 -j MASQUERADE")
 
    CLI( net )
 
    print "*** Stopping network"
    net.stop()
 
if __name__ == '__main__':
    setLogLevel( 'info' )
    topology()

```
### 執行 test.py
### mininet > xterm h1 h1 h2 h3
### h1(1) > echo hi > hi.htm
### h1(1) > python -m SimpleHTTPServer 80
### h2 > cd frp/conf
### h2 > cat frps.ini
![20220411-7](/img/20220411-7.jpg)
### h2 > ./frps -c frps.ini
![20220411-8](/img/20220411-8.jpg)
### h1(2) > cd frp/conf
### h1(2) > cat frpc.ini
![20220411-9](/img/20220411-9.jpg)
### h1(2) > ./frpc -c frpc.ini
![20220411-10](/img/20220411-10.jpg)
### h3 > cat /etc/hosts
![20220411-11](/img/20220411-11.jpg)
### h3 > curl www.example.com:8080/hi.htm 
![20220411-12](/img/20220411-12.jpg)

----

### ssh tunnel
用來保護傳統沒有加密的資料可以透過 ssh tunnel 傳輸
### cd /home/user/server-test/test-sshtunnel
### 編輯 1.py
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
    d1 = net.addDocker('d1', ip='192.168.0.2/24', dimage="ubuntu:1.0")
 
    info('*** Creating links\n')
    net.addLink(h1, d1)
   
    info('*** Starting network\n')
    net.start()
    #d1.cmd("/etc/init.d/ssh start")
    #d1.cmd("/etc/init.d/apache2 start")
    #h1.cmd("ssh -Nf -L 192.168.0.1:5555:192.168.0.2:80 user@192.168.0.2")   
 
    info('*** Running CLI\n')
    CLI(net)
 
    info('*** Stopping network')
    net.stop()
 
if __name__ == '__main__':
    setLogLevel('info')
    topology()
```
### 執行 1.py
![20220411-13](/img/20220411-13.jpg)
### containernet > xterm h1 h1
### 另開終端機，啟動docker，啟動 ssh 、 apache2
![20220411-14](/img/20220411-14.jpg)
### h1(1) > wireshark 
### h1(2) > curl 192.168.0.2/hi.htm
### 資料以明文傳輸
![20220411-15](/img/20220411-15.jpg)
### 透過 ssh tunnel 傳輸
### h1(2) > ssh -Nf -L 192.168.0.1:5555:192.168.0.2:80 root@192.168.0.2
![20220411-16](/img/20220411-16.jpg)
### 本地端已建立 shh tunnel
![20220411-17](/img/20220411-17.jpg)
### h1(2) > curl 192.168.0.1:5555/hi.htm
### 資料已加密
![20220411-18](/img/20220411-18.jpg)
