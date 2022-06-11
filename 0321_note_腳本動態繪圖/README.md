## 使用腳本動態繪圖
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
- xterm h1 h2 h2 h2 
### h2(1) 執行 process.sh
```
filename='a'
> result                       
> a
b="not"
while true                       # 讀取吞吐量, 進行過濾
do
   if [ -s "$filename" ]         # 判斷檔案是否為空
   then
        #echo "$filename is NOT empty file."
        while IFS= read -r line
        do
          result=`echo "$line" | grep "sec"`
          if [[ -n $result ]]
          then
            #echo $result
            b="done"
            break
          fi
        done < a
   fi
 
   if [ $b = "done" ]
   then
     break
   fi
done
 
while IFS= read -r line
do
  result=`echo "$line" | grep "sec" | tr "-" " " | awk '{print $4,$8}'`
  if [[ -n $result ]]
  then
    echo $result
    echo $result >> result
    sleep 1
  fi
done < a
```
### h2(2) 執行 plot-throughput.sh
```
filename='result'
while true                
do
   if [ -s "$filename" ]  # 判斷 result 裡面是否有東西存在
   then
        #echo "$filename is NOT empty file."
        break
   fi
done
 
gnuplot gnuplot-plot

```
### 畫圖的程式 gnuplot-plot
```
FILE = 'result'
stop = 0
 
N = 10
set yrange [0:100]
set ytics 0,10,100
set key off
set xlabel "time(sec)"
set ylabel "Throughput(Mbps)"
 
while (!stop) {  
    pause 0.1       # pause in seconds
    stats [*:*][*:*] FILE u 0 nooutput
    lPnts=STATS_records<N ? 0: STATS_records-N
    plot FILE u 1:2 every ::lPnts w lp pt 7   
    # u 為使用 1 2 行
}

```
### h2(3) 執行 > iperf -s -i 1 > a 
### h1 執行 iperf -c 192.168.2.1 -t 100
### 結果
![20220321-1](/img/20220321-1.jpg)
