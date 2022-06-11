## FFMPEG 實驗
### 安裝 FFMPEG 
- apt install ffmpeg
### FFMPEG 可用來做影像的壓縮、影像的串流和影像的接收。
## 影像的壓縮encode 解碼decode 串流stream
### 下載影像網址
- https://media.xiph.org/video/derf/
### 傳換 yuv 格式 ( 把RGB三元素轉換為亮度、色度 )
- ffmpeg -i foreman_cif.y4m foreman_cif.yuv
### 進行影像壓縮
- ffmpeg -f rawvideo -s:v 352x288 foreman_cif.yuv -c:v libx264 -qp 30 -g 12 -bf 2 -f mpeg foreman.mp4(-f:指定yuv為原始格式rawvideo 指定格式352x288 每秒撥放30 使用壓縮格式libx264 -qp:qp值越大,壓縮比越高)

![20220328-1](/img/20220328-1.jpg)
### 播放 foreman.mp4
- ffplay foreman.mp4

![20220328-2](/img/20220328-2.jpg)
### 比較影像的差異 - psnr
- 參考：[psnr](https://zh.wikipedia.org/zh-tw/%E5%B3%B0%E5%80%BC%E4%BF%A1%E5%99%AA%E6%AF%94)
### psnr 越高, 失真越小, psnr 越低, 失真越大
### psnr.c 程式碼
```c
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

int main(int n, char *cl[])
{
  FILE *f1, *f2;
  int i, x, y, yuv, inc = 1, size = 0, N = 0, Y, F;
  double yrmse, diff, mean = 0, stdv = 0, *ypsnr = 0;
  unsigned char *b1, *b2;
  int k=1;
  clock_t t = clock();

  if (n != 6 && n != 7) {
    puts("psnr x y <YUV format> <src.yuv> <dst.yuv> [multiplex]");
    puts("  x\t\tframe width");
    puts("  y\t\tframe height");
    puts("  YUV format\t420, 422, etc.");
    puts("  src.yuv\tsource video");
    puts("  dst.yuv\tdistorted video");
    puts("  [multiplex]\toptional");
    return EXIT_FAILURE;
  }

  if ((f1 = fopen(cl[4],"rb")) == 0) goto A;
  if ((f2 = fopen(cl[5],"rb")) == 0) goto B;
  if (!(x = strtoul(cl[1], 0, 10)) ||
      !(y = strtoul(cl[2], 0, 10))) goto C; 
  if ((yuv = strtoul(cl[3], 0, 10)) > 444) goto D;
  if (cl[6] && !strcmp(cl[6], "multiplex")) inc = 2;

  Y = x * y;
  switch (yuv) {
    case 400: F = Y; break;
    case 422: F = Y * 2; break;
    case 444: F = Y * 3; break;
    default :
    case 420: F = Y * 3 / 2; break;
  }

  if (!(b1 = malloc(F))) goto E;
  if (!(b2 = malloc(F))) goto E;

  for (;;) {
    if (1 != fread(b1, F, 1, f1) || 1 != fread(b2, F, 1, f2)) break;
    for (yrmse=0, i=inc-1; i<(inc==1 ? Y : F); i+=inc) {
      diff = b1[i] - b2[i];
      yrmse += diff * diff;
    }
    if (++N > size) {
      size += 0xffff;
      if (!(ypsnr = realloc(ypsnr, size * sizeof *ypsnr))) goto E;
    }

    mean += ypsnr[N-1] = yrmse ? 20 * (log10(255 / sqrt(yrmse / Y))) : 0;
    printf("%d\t%.2f\n", k++, ypsnr[N-1]);
  }

  if (N) {
    mean /= N;

    for (stdv=0, i=0; i<N; i++) {
      diff = ypsnr[i] - mean;
      stdv += diff * diff;
    }
    stdv = sqrt(stdv / (N - 1));

    free(ypsnr);
  }

  fclose(f1);
  fclose(f2);

  //fprintf(stderr, "psnr:\t%d frames (CPU: %lu s) mean: %.2f stdv: %.2f\n",
  //  N, (unsigned long) ((clock() - t) / CLOCKS_PER_SEC), mean, stdv);

  return 0;

A: fprintf(stderr, " Error opening sourcefile.\n"); goto X;
B: fprintf(stderr, " Error opening decodedfile.\n"); goto X;
C: fprintf(stderr, " Invalid width or height.\n"); goto X;
D: fprintf(stderr, " Invalid YUV format.\n"); goto X;
E: fprintf(stderr, " Not enough memory.\n");

X: return EXIT_FAILURE;
}

```
### 編譯 psnr.c
- gcc -o psnr psnr.c -lm
### 解壓縮 foreman.mp4 、 foreman2.mp4 還原成 yuv
- ffmpeg -i foreman.mp4 1.yuv
![20220328-3](/img/20220328-3.jpg)
### 執行 psnr , 比較 foreman_cif yuv 1.yuv 
- ./psnr 352 288 420 foreman_cif.yuv 1.yuv > psnr1 (420為格式)
### 執行 psnr , 比較 foreman_cif yuv 2.yuv 
- ./psnr 352 288 420 foreman_cif.yuv 2.yuv > psnr2 
### 繪製成圖片
![20220328-4](/img/20220328-4.jpg)
### 執行 0328-1.py ,比較不同遺失率的結果
### 遺失率為 0%
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
- xterm h1 h2 
### h2 執行 ffplay -i udp://192.168.2.1:1234
### h1 執行串流 ffmpeg -re -i foreman.mp4 -c copy -f mpegts udp://192.168.2.1:1234
![20220328-5](/img/20220328-5.jpg)
### 遺失率改為 1% , 結果較為模糊
![20220328-6](/img/20220328-6.jpg)
## 三組不同遺失率比較
### 第一組:遺失率為0%
```
  h1r = {'bw':100,'delay':'1ms','loss':0}  
  net.addLink(h1, r, cls=TCLink , **h1r)
  h2r = {'bw':100,'delay':'1ms','loss':0}     
```
### h2 > ffmpeg -i udp://192.168.2.1 -c copy 2-0.ts
### h1 > ffmpeg -re -i foreman2.mp4 -c copy -f mpegts udp://192.168.2.1:1234
### h2 > ffmpeg -i 2-0.ts 2-0.yuv 
### h2 > ./psnr 352 288 420 foreman_cif.yuv 2-0.yuv > psnr2-0
### 第二組:遺失率為1%
```
  h1r = {'bw':100,'delay':'1ms','loss':0}  
  net.addLink(h1, r, cls=TCLink , **h1r)
  h2r = {'bw':100,'delay':'1ms','loss':1}     
```
### h2 > ffmpeg -i udp://192.168.2.1 -c copy 2-1.ts
### h1 > ffmpeg -re -i foreman2.mp4 -c copy -f mpegts udp://192.168.2.1:1234
### h2 > ffmpeg -i 2-1.ts 2-1.yuv 
### h2 > ./psnr 352 288 420 foreman_cif.yuv 2-1.yuv > psnr2-1
### 第三組:遺失率為3%
```
  h1r = {'bw':100,'delay':'1ms','loss':0}  
  net.addLink(h1, r, cls=TCLink , **h1r)
  h2r = {'bw':100,'delay':'1ms','loss':3}     
```
### h2 > ffmpeg -i udp://192.168.2.1 -c copy 2-3.ts
### h1 > ffmpeg -re -i foreman2.mp4 -c copy -f mpegts udp://192.168.2.1:1234
### h2 > ffmpeg -i 2-3.ts 2-3.yuv 
### h2 > ./psnr 352 288 420 foreman_cif.yuv 2-3.yuv > psnr2-3
### 繪圖結果
![20220328-7](/img/20220328-7.jpg)
