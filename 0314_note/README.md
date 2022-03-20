## miniedit - 圖形化介面工具
### 執行 miniedit.py
```
cd mininet/examples
./miniedit.py
```
### 環境
- ![20220314-1](/img/20220314-1.jpg)
### 線路設定
- ![20220314-2](/img/20220314-2.jpg)
### 輸出成 py 檔
```py
#!/usr/bin/env python

from mininet.net import Mininet
from mininet.node import Controller, RemoteController, OVSController
from mininet.node import CPULimitedHost, Host, Node
from mininet.node import OVSKernelSwitch, UserSwitch
from mininet.node import IVSSwitch
from mininet.cli import CLI
from mininet.log import setLogLevel, info
from mininet.link import TCLink, Intf
from subprocess import call

def myNetwork():

    net = Mininet( topo=None,
                   build=False,
                   ipBase='10.0.0.0/8')

    info( '*** Adding controller\n' )
    info( '*** Add switches\n')
    r1 = net.addHost('r1', cls=Node, ip='0.0.0.0')
    r1.cmd('sysctl -w net.ipv4.ip_forward=1')

    info( '*** Add hosts\n')
    h2 = net.addHost('h2', cls=Host, ip='10.0.0.2', defaultRoute=None)
    h1 = net.addHost('h1', cls=Host, ip='10.0.0.1', defaultRoute=None)

    info( '*** Add links\n')
    h1r1 = {'bw':100,'delay':'1ms','loss':0}
    net.addLink(h1, r1, cls=TCLink , **h1r1)
    h2r1 = {'bw':10,'delay':'1ms','loss':0}
    net.addLink(h2, r1, cls=TCLink , **h2r1)

    info( '*** Starting network\n')
    net.build()
    info( '*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info( '*** Starting switches\n')

    info( '*** Post configure switches and hosts\n')

    CLI(net)
    net.stop()

if __name__ == '__main__':
    setLogLevel( 'info' )
    myNetwork()

```
## 效能量測工具 iperf
### 安裝 iperf
```
apt install iperf
```
### 環境
- ![20220314-3](/img/20220314-3.jpg)
### 執行 3.py
```py
#!/usr/bin/python

from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  r=net.addHost('r')
  
  Link(h1,r)
  Link(h2,r)
  net.build()

  h1.cmd("ifconfig h1-eth0 0")
  h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")
  h1.cmd("ip route add default via 192.168.1.254")
  h2.cmd("ifconfig h2-eth0 0")
  h2.cmd("ip a a 192.168.2.1/24 brd + dev h2-eth0")
  h2.cmd("ip route add default via 192.168.2.254")

  r.cmd("ifconfig r-eth0 0")
  r.cmd("ifconfig r-eth1 0")
  r.cmd("ip a a 192.168.1.254/24 brd + dev r-eth0")
  r.cmd("ip a a 192.168.2.254/24 brd + dev r-eth1")
  r.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
  CLI(net)
  net.stop()
```
### 每秒鐘吞吐量多少
- h2> iperf -s -i 1 ( s 為 server )
### 傳給伺服器十秒
- h1> iperf -c 192.168.2.1 -t 10 ( c 為 client )
![20220314-4](/img/20220314-4.jpg)
### 兩條線路並限制速度, 使用UDP傳輸, 不同埠號區隔
![20220314-5](/img/20220314-5.jpg)
## 使用畫圖軟體
### 安裝
```
apt install gnuplot
```
### 加入線路參數 3.py
```py
#!/usr/bin/python

from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  r=net.addHost('r')
  h1r = {'bw':100,'delay':'1ms','loss':0}  
  net.addLink(h1, r, cls=TCLink , **h1r)
  h2r = {'bw':100,'delay':'1ms','loss':0}
  net.addLink(h2, r, cls=TCLink , **h2r)
  Link(h1,r)
  Link(h2,r)
  net.build()

  h1.cmd("ifconfig h1-eth0 0")
  h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")
  h1.cmd("ip route add default via 192.168.1.254")
  h2.cmd("ifconfig h2-eth0 0")
  h2.cmd("ip a a 192.168.2.1/24 brd + dev h2-eth0")
  h2.cmd("ip route add default via 192.168.2.254")

  r.cmd("ifconfig r-eth0 0")
  r.cmd("ifconfig r-eth1 0")
  r.cmd("ip a a 192.168.1.254/24 brd + dev r-eth0")
  r.cmd("ip a a 192.168.2.254/24 brd + dev r-eth1")
  r.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
  CLI(net)
  net.stop()
```
![20220314-9](/img/20220314-9.jpg)
### 把檔案導入 result
- iperf -s -i 1 > result or iperf -s -i 1 | tee result

![20220314-10](/img/20220314-10.jpg)
### 簡化
```
cat result | grep "sec" | head -n 10 | tr "-" " " | awk '{print $4,$8}' > tcp_result
```
### 呼叫 gnuplot
![20220314-11](/img/20220314-11.jpg)
![20220314-12](/img/20220314-12.jpg)
- result_udp.gif
![20220314-13](/img/20220314-13.jpg)
### UDP 與 TCP 同時跑的情形
- h2(1)> iperf -s -i 1 -p 5555 | tee tcp
- h2(2)> iperf -s -i 1 -u -p 6666 | tee udp
- h1(2)> iperf -c 192.168.2.1 -u -b 50M -t 30 -p 6666
- h1(1)> iperf -c 192.168.2.1 -t 50 -p 5555
![20220314-14](/img/20220314-14.jpg)
### 把結果存放到 tcp_result udp_result
- h2(1)> cat tcp | grep "sec" |head -n 50 | tr "-" " " | awk '{print $4,$8}' > tcp_result
- h2(2)> cat udp | grep "sec" | grep -v out-of-order | tr "-" " " | head -n 30 | awk '{print $4,$8}' > udp_result
### 使用腳本輸出成圖片
![20220314-15](/img/20220314-15.jpg)

## 安裝 Ettercap
### 資料:
- 參考：[Ettercap安装教程_yanhanhui1的博客-CSDN博客_ettercap安装](https://blog.csdn.net/yanhanhui1/article/details/104960378)
### 進入ettercap官網，下載僅原始程式碼版本
- ETTERCAP : [官網](https://www.ettercap-project.org/downloads.html)
### 解壓
![20220314-6](/img/20220314-6.jpg)
### 想編譯成功ettercap，我們必須安裝一些依賴庫，執行以下命令安裝：
```
sudo apt-get install debhelper bison check cmake flex ghostscript libbsd-dev libcurl4-openssl-dev libgeoip-dev libltdl-dev libluajit-5.1-dev libncurses5-dev libnet1-dev libpcap-dev libpcre3-dev libssl-dev libgtk-3-dev libgtk2.0-dev
```
### 進入解壓后的ettercap資料夾，執行下列命令：
```
mkdir build 
cd build
sudo cmake ../
sudo make install 
```
![20220314-7](/img/20220314-7.jpg)
### 執行 ettercap -G
![20220314-8](/img/20220314-8.jpg)
### 環境 - h3欺騙交換機偷取 h1 傳給 h2 的封包
![20220314-16](/img/20220314-16.jpg)
### 執行1.py
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
  br=net.addHost('br')
  Link(h1,br)
  Link(h2,br)
  Link(h3,br)
  net.build()
  h1.cmd("ifconfig h1-eth0 0")
  h2.cmd("ifconfig h2-eth0 0")
  h3.cmd("ifconfig h3-eth0 0")
  h1.cmd("ifconfig br-eth0 0")
  h2.cmd("ifconfig br-eth1 0")
  h3.cmd("ifconfig br-eth2 0")
  br.cmd("brctl addbr mybr")
  br.cmd("brctl addif mybr br-eth0")
  br.cmd("brctl addif mybr br-eth1")
  br.cmd("brctl addif mybr br-eth2")
  br.cmd("ifconfig mybr up")

  h1.cmd("ip a a 192.168.10.1/24 dev h1-eth0")
  h2.cmd("ip a a 192.168.10.2/24 dev h2-eth0")
  h3.cmd("ip a a 192.168.10.3/24 dev h3-eth0")
  h1.cmd("ifconfig h1-eth0 hw ether 00:00:00:00:00:01")
  h2.cmd("ifconfig h2-eth0 hw ether 00:00:00:00:00:02")
  h3.cmd("ifconfig h3-eth0 hw ether 00:00:00:00:00:03")
  CLI(net)
  net.stop()
```
- mininet> xterm h1 h2 h3
- h3> wireshark
- h1> ping 10.0.0.2
- h3> ettercap -G
### 掃描網路上有哪些主機
![20220314-17](/img/20220314-17.jpg)
### Hosts list
![20220314-18](/img/20220314-18.jpg)
![20220314-19](/img/20220314-19.jpg)
- h1 設為 target1
- h2 設為 target2
### ARP poisoning
![20220314-20](/img/20220314-20.jpg)
### h3 wireshark 即可聽到
![20220314-21](/img/20220314-21.jpg)
### h1 使用靜態綁定
- h1> arp -s 192.168.10.2 00:00:00:00:00:02
- h2> arp -s 192.168.10.1 00:00:00:00:00:01
- h1 ping h2 , h3 即聽不到
### 無法騙主機 , 開始騙交換機 - port stealing
![20220314-22](/img/20220314-22.jpg)
![20220314-23](/img/20220314-23.jpg)
### 結果
![20220314-24](/img/20220314-24.jpg)
### 防範 switch port 寫死 網路卡卡號