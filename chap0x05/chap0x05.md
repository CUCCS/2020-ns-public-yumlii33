**目录**

<a herf = "标题">基于Scapy编写端口扫描器</a>

<a herf="实验目的">实验目的</a>

<a herf="实验环境">实验环境</a>

<a herf="实验要求">实验要求</a>

<a herf="实验步骤">实验步骤</a>

<a herf="0">0 主机状态设置</a>

[1 TCP connect scan](#1-tcp-connect-scan)

[2 TCP stealth scan](#2-tcp-stealth-scan)

<a herf="3"></a>

<a herf="4"></a>

<a herf="5"></a>

<a herf="6"></a>

<a herf="实验总结">实验总结</a>







# <a id="标题">基于Scapy编写端口扫描器</a>

## <a id="实验目的">实验目的</a>

- 掌握网络扫描之端口状态探测的基本原理

## <a id="实验环境">实验环境</a>

- `python` + [`scapy`](https://scapy.net/)

- 局域网网络拓扑图

  ![NS-scan-network](/img/NS-scan-network.png)

## <a id="实验要求">实验要求</a>

- 禁止探测互联网上的 `IP` ，严格遵守网络安全相关法律法规
- 完成以下扫描技术的编程实现
  - `TCP connect scan / TCP stealth scan`
  - `TCP Xmas scan / TCP fin scan / TCP null scan`
  - `UDP scan`
- 上述每种扫描技术的实现测试均需要测试端口状态为：`开放`、`关闭` 和 `过滤` 状态时的程序执行结果
- 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因
- 在实验报告中详细说明实验网络环境拓扑、被测试`IP`的端口状态是如何模拟的
- （可选）复刻 `nmap` 的上述扫描技术实现的命令行参数开关

## <a id="实验步骤">实验步骤</a>

### 0 主机状态设置

端口不是独立存在的，它是依附于进程的。某个进程开启，那么它对应的端口就开启了，进程关闭，则该端口也就关闭了。

* `Open`状态

  ```
  # 开启apache服务，以启用端口80
  service apache2 start
  
  # 查看开启状态
  netstat -ntulp | grep 80 
  ```

  ![open80port](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\open80port.png)

* `Close`状态

  ```
  # 关闭apache服务，以关闭端口80
  service apache2 stop
  
  # 查看80端口对应的进程
  lsof -i:80
  ```

  ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\close80port.png)
  
* `Filtered`状态

  ```
  # 安装 ufw 防火墙（Kali 2019 没有自带防火墙）
  apt-get install ufw 
  
  # 查看防火墙状态（默认 inactive）
  ufw status
  
  # 开启apache服务的同时开启防火墙，模拟filtered状态
  ufw enable
  
  # 关闭防火墙
  ufw disable
  ```

### <a id="1">1 `TCP connect scan`</a>

#### 1.1 代码

* [tcpConnectScan.py](/code/tcpConnectScan.py)

#### 1.2 测试开放端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpConnectScan-open-Scapy.png)

  * `nmap` 扫描    (`nmap -p80 -sT 172.16.111.113`)

    ![tcpConnectScan-open-Nmap](/img/tcpConnectScan-open-Nmap.png)

* 数据包分析

  * [`tcpConnectScan-open.pcap`]()

  * `scapy` 扫描
  * `nmap`扫描

#### 1.3 测试关闭端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpConnectScan-closed-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpConnectScan-closed-Nmap.png)

* 数据包分析

  * [`tcpConnectScan-closed.pcap`]()
  * `scapy`扫描
  * `nmap`扫描

#### 1.4 测试过滤端口

* 扫描过程

  * `Scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpConnectScan-filtered-Nmap.png)

  * `Nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpConnectScan-filtered-Nmap.png)

* 数据包分析

  * [`tcpConnectScan-filtered.pcap`]()
  * `scapy`扫描
  * `nmap`扫描

### 2 `TCP stealth scan`

#### 2.1 代码

* [tcpStealthScan.py]()

#### 2.2 测试开放端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpStealthScan-open-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpStealthScan-open-Nmap.png)

* 数据包分析

  * [`tcpStealthScan-open.pcap`]()
  * `scapy`扫描
  * `nmap`扫描

#### 2.3 测试关闭端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpStealthScan-closed-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpStealthScan-closed-nmap.png)

* 数据包分析

  * [`tcpStealthScan-close.pcap`]()

#### 2.4 测试过滤端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpStealthScan-filtered-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpStealthScan-filtered-Nmap.png)

* 数据包分析
  
  * [`tcpStealthScan-filtered.pcap`]()

### 3 `TCP Xmas scan` 

#### 3.1 代码

* [tcpXmasScan.py](/code/tcpXmasScan.py)

#### 3.2 测试开放端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpXmasScan-open-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpXmasScan-open-Nmap.png)

* 数据包分析

  * [tcpXmasScan-open.pcap]()

  * `scapy`扫描

    

  * `nmap`扫描

    

#### 3.3 测试关闭端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpXmasScan-closed-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpXmasScan-closed-Nmap.png)

* 数据包分析

  * [tcpXmasScan-closed.pcap]()
  * `scapy`扫描
  * `nmap`扫描

#### 3.4 测试过滤端口

* 扫描过程

  * `scapy`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpXmasScan-filtered-Scapy.png)

  * `nmap`扫描

    ![](D:\Project_NetworkSecurityProjects\2020-ns-public-yumlii33\chap0x05\img\tcpXmasScan-filtered-Nmap.png)

* 数据包分析

  * [tcpXmasScan-filtered.pcap]()
  * `scapy`扫描
  * `nmap`扫描

### 4 `TCP fin scan`

#### 4.1 代码

* [tcpFinScan.py]()

#### 4.2 测试开放端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

#### 4.3 测试关闭端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

#### 4.4 测试过滤端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

### 5 `TCP null scan`

#### 5.1 代码

* [tcpNullScan.py]()

#### 5.2 测试开放端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

#### 5.3 测试关闭端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

#### 5.4 测试过滤端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

### 6 `UDP scan`

#### 6.1 代码

* [udpScan.py]()

#### 6.2 测试开放端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

#### 6.3 测试关闭端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

#### 6.4 测试过滤端口

* 扫描过程
  * `scapy`扫描
  * `nmap`扫描
* 数据包分析
  * []()
  * `scapy`扫描
  * `nmap`扫描

## 实验总结





## Q&A

* Q1：如何关闭端口？

  A：端口不是独立存在的，它是依附于进程的。某个进程开启，那么它对应的端口就开启了，进程关闭，则该端口也就关闭了。下次若某个进程再次开启，则相应的端口也再次开启。而不要纯粹的理解为关闭掉某个端口，不过可以禁用某个端口。

* Q2：如果先进行开放端口扫描，然后关闭进程再次扫描的结果是`Flitered`，如果想得到`Closed`的扫描结果，可以重启目标主机。

* Q3：





## 参考资料

* [第五章网络扫描在线课件](https://c4pr1c3.github.io/cuc-ns-ppt/chap0x05.md.html)
* []()

* [查看端口使用状态、关闭端口方法（netstat,lsof）](https://www.cnblogs.com/alantu2018/p/8462574.html)



## 补充：课后问答题

1. 通过本章网络扫描基本原理的学习，试推测
   - 应用程序版本信息扫描原理
   - 网络漏洞扫描原理

2. 网络扫描知识库的构建方法有哪些？

3. 除了 `nmap` 之外，目前还有哪些流行的网络扫描器？和 `nmap` 进行优缺点对比分析



