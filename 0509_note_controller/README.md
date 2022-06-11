### ex1.使用 pox 控制器下放規則到 s1 
### ![20220509-1](/img/20220509-1.jpg)
- test.py
```py
#!/usr/bin/python

from mininet.net import Mininet
from mininet.node import RemoteController, OVSKernelSwitch, Host
from mininet.cli import CLI
from mininet.link import TCLink, Intf
from mininet.log import setLogLevel, info
from subprocess import call


def myNetwork():

    ##net = Mininet(topo=None, build=False, ipBase='10.0.0.0/8')
    net = Mininet()

    info( '*** Adding controller\n' )
    c0 = net.addController(name='c0',
                           controller=RemoteController,
                           ip='127.0.0.1',
                           protocol='tcp',
                           port=6633)

    info( '*** Add switches/APs\n')
    s1 = net.addSwitch('s1', cls=OVSKernelSwitch)

    info( '*** Add hosts/stations\n')
    h2 = net.addHost('h2', cls=Host, ip='192.168.1.2/24', mac='00:00:00:00:00:02', defaultRoute=None)
    h1 = net.addHost('h1', cls=Host, ip='192.168.1.1/24', mac='00:00:00:00:00:01', defaultRoute=None)

    info( '*** Add links\n')
    net.addLink(h1, s1)
    net.addLink(s1, h2)

    info( '*** Starting network\n')
    net.build()
    info( '*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info( '*** Starting switches/APs\n')
    net.get('s1').start([c0])

    info( '*** Post configure nodes\n')

    CLI(net)
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    myNetwork()

```
### cd /home/user/pox
### ./pox.py forwarding.hub
### 關閉 pox 控制器也可互相通訊 ,規則已下放
### ex2.
### 執行 test1.py
### mininet> sh ovs-ofctl show s1 
### ![20220509-2](/img/20220509-2.jpg)
### capabilities::
- FLOW_STATS : 規則統計
- PORT_STATS : 埠號統計
### actions :
- enqueue : 設定佇列
- set_vlan_vid、set_vlan_pcp、strip_vlan : vlan 處理 , 加入移除標籤
- mod_dl_src、mod_dl_dst : modify datalink src 、 modify datalink src 
- mod_nw_src、mod_nw_dst、mod_nw_tos : modify network src 、 modify network src 
- mod_tp_src、mod_tp_dst : 修改第四層 Transport Layer
### cd /home/user/pox
### ./pox.py forwarding.l2_learning 
### 觀察規則 
### ![20220509-3](/img/20220509-3.jpg)
- idle_timeout=10 : 閒置一定時間後刪除
- hard_timeout=30 : 超過時間後刪除
- icmp : icmp
- in_port : 來源埠號
- dl_src 、 dl_dst : 來源 、 目的 mac
- nw_src 、 nw_dst : 來源 、 目的 ip
- actions=output:1 : 傳送至1號埠
- icmp_type=0 : 
![20220509-4](/img/20220509-4.jpg)
### 規則由底層開始寫入 
- in_port > mac > ip > tcp
### 執行 test1.py
### arp 手動寫入
### mininet> h1 arp -s 192.168.1.2 00:00:00:00:00:02
### mininet> h2 arp -s 192.168.1.1 00:00:00:00:00:01
### mininet> sh ovs-ofctl add-flow s1 ip,dl_dst=00:00:00:00:00:02,nw_dst=192.168.1.2,actions=output:2
### mininet> sh ovs-ofctl add-flow s1 dl_type=0x0800,dl_dst=00:00:00:00:00:01,nw_dst=192.168.1.1,actions=output:1
### mininet> h1 ping h2
### ex3.
### ![20220509-5](/img/20220509-5.jpg)
- test2.py
```py
#!/usr/bin/python

from mininet.net import Mininet
from mininet.node import RemoteController, OVSKernelSwitch, Host
from mininet.cli import CLI
from mininet.link import TCLink, Intf
from mininet.log import setLogLevel, info
from subprocess import call


def myNetwork():

    net = Mininet()

    info( '*** Adding controller\n' )
    c0 = net.addController(name='c0',
                           controller=RemoteController,
                           ip='127.0.0.1',
                           protocol='tcp',
                           port=6633)

    info( '*** Add switches/APs\n')
    s1 = net.addSwitch('s1', cls=OVSKernelSwitch)
    s2 = net.addSwitch('s2', cls=OVSKernelSwitch)

    info( '*** Add hosts/stations\n')
    h1 = net.addHost('h1', cls=Host, ip='10.0.0.1/24', defaultRoute=None)
    h2 = net.addHost('h2', cls=Host, ip='10.0.0.2/24', defaultRoute=None)

    info( '*** Add links\n')
    net.addLink(h1, s1)
    net.addLink(h2, s2)
    net.addLink(s1, s2)

    info( '*** Starting network\n')
    net.build()
    info( '*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info( '*** Starting switches/APs\n')
    net.get('s1').start([c0])
    net.get('s2').start([c0])

    info( '*** Post configure nodes\n')

    CLI(net)
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    myNetwork()

```
### 執行 test2.py
### mininet> sh ovs-ofctl add-flow s1 arp,actions=flood
### mininet> sh ovs-ofctl add-flow s2 arp,actions=flood
### mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.1,action=output:1
### mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.2,action=output:2
### mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.1,action=output:2
### mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.2,action=output:1

