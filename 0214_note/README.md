## 虛擬環境架設
- ![eve_login](/img/eve_login.jpg)
### ip:192.168.52.136
- ![eve_web](/img/eve_web.jpg)
## 小實驗:
- ![0914test1](/img/0914test1.jpg)
### 開啟VPC1 VPC3 並設置IP
```
VPC1:
ip 192.168.1.1 255.255.255.0
VPC3:
ip 192.168.1.2 255.255.255.0
```
### VPC3 ping VPC1
```
ping 192.168.1.1
```
### VPC1 ping VPC3
```
ping 192.168.1.2
```
- ![0914test1_result](/img/0914test1_result.jpg)
