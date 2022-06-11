## 使用環境 Ubuntu 16.04
- 安裝 Ubuntu 16.04：[Ubuntu 16.04](https://releases.ubuntu.com/16.04/ubuntu-16.04.7-desktop-amd64.iso)
## 安裝 Mininet
- git clone https://github.com/mininet/mininet.git
- cd mininet 
- util/install.sh -a
## 使用 mininet 最簡單指令
- mn : 架設 mininet
- mininet> xterm : 創建終端機
## 課堂練習
- mininet> xterm h1 h2 
### h2 新增 hi.htm
- echo "hi" > hi.htm
### h2 啟動網頁伺服器
- python -m SimpleHTTPServer 80
### h1 抓取網頁內容
- curl http://10.0.0.2/hi.htm
### 結果
- ![20220214](/img/20220214.jpg)
### 資料:
- 參考：[mininet](https://github.com/mininet/mininet.git)