## mininet 使用 network namespace 的隔離
### 相關操作指令
- ip netns add xx 創建一个 namespace
- ip netns exec xx yy 在新 namespace xx 中執行 yy 命令
- ip link add type veth 創建 veth pair # 創造虛擬網路卡 , 創造虛擬網路卡 , 為一對
- ip link set xx netns yy 將 veth xx 加入到 namespace yy 中
### 環境
- ![20220214](/img/20220221-1.jpg)

### 使用ip link 和 brctl 建立 bridge
- apt install bridge-utils
- ip link add br0 type bridge 
- ip link set dev br0 up
- brctl show  
### 建立 veth pair
- ip link add type veth
- ip link add type veth
- ip link add type veth
### 創建三個 network space
- ip netns add net0
- ip netns add net1
- ip netns add net2
### 將 veth pair 的一頭掛到 namespace 中，一頭掛到 bridge 上，並設 IP 位址
// （1）配置第 1 个 net0
- ip link set dev veth1 netns net0
- ip netns exec net0 ip link set dev veth1 name eth0
- ip netns exec net0 ip addr add 10.0.1.1/24 dev eth0
- ip netns exec net0 ip link set dev eth0 up
- ip link set dev veth0 master br0
- ip link set dev veth0 up

// （2）配置第 2 个 net1
- ip link set dev veth3 netns net1
- ip netns exec net1 ip link set dev veth3 name eth0
- ip netns exec net1 ip addr add 10.0.1.2/24 dev eth0
- ip netns exec net1 ip link set dev eth0 up
- ip link set dev veth2 master br0
- ip link set dev veth2 up

// （3）配置第 3 个 net2
- ip link set dev veth5 netns net2
- ip netns exec net2 ip link set dev veth5 name eth0
- ip netns exec net2 ip addr add 10.0.1.3/24 dev eth0
- ip netns exec net2 ip link set dev eth0 up
- ip link set dev veth4 master br0
- ip link set dev veth4 up
### 將 veth0 veth2 veth4掛載到br0上
- ip link set dev veth0 master br0
- ip link set dev veth2 master br0
- ip link set dev veth4 master br0
- brctl show
- ip link set dev veth0 up
- ip link set dev veth2 up
- ip link set dev veth4 up
- ifconfig 
- ip netns exec net0 ping 10.0.1.2 -c 3 
- ip netns exec net0 ping 10.0.1.3 -c 3
### 刪除配置
- brctl delif br0 veth0
- ip link delete veth0
- ifconfig br0 down
- brctl delbr br0 
- ifconfig
- ip netns del net0
- ip netns ls

## mininet 基礎
### 使用 mn 指令 產生環境
- ![20220221-2](/img/20220221-2.jpg)
- mininet> net 查看整個網路架構
### 使用 mn --topo linear,3 指令 產生環境
- ![20220221-3](/img/20220221-3.jpg)
### 使用 mn --topo tree,2 指令 產生環境
- ![20220221-4](/img/20220221-4.jpg)
### 資料:
- 參考：[详解云计算网络底层技术——Linux network namespace 原理与实践](https://segmentfault.com/a/1190000018391069)
- 參考：[容器化技術的網路難題，為什麼它是安全的?  - 德鴻科技 Grandsys](https://www.grandsys.com.tw/news/rd/901-linux-docker)