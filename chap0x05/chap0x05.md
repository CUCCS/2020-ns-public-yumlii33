**目录**

[基于Scapy编写端口扫描器](#基于scapy编写端口扫描器)

* [实验目的](#实验目的)

* [实验环境](#实验环境)

* [实验要求](#实验要求)

* [实验步骤](#实验步骤)
  * [0 主机状态设置](#0-主机状态设置)
  * [1 TCP connect scan](#1-tcp-connect-scan)
  * [2 TCP stealth scan](#2-tcp-stealth-scan)
  * [3 TCP Xmas scan](#3-tcp-xmas-scan)
  * [4 TCP fin scan](#4-tcp-fin-scan)
  * [5 TCP null scan](#5-tcp-null-scan)
  * [6 UDP scan](#6-udp-scan)

* [实验总结](#实验总结)
* [Q&A](#Q&A)
* [参考资料](#参考资料)
* [课后问答题](#课后问答题)

# 基于Scapy编写端口扫描器

## 实验目的

- 掌握网络扫描之端口状态探测的基本原理

## 实验环境

- `python` + [`scapy`](https://scapy.net/)

- 局域网网络拓扑图

  ![NS-scan-network](/chap0x05/img/NS-scan-network.png)

## 实验要求

- [x] 禁止探测互联网上的 `IP` ，严格遵守网络安全相关法律法规
- [x] 完成以下扫描技术的编程实现
  - `TCP connect scan / TCP stealth scan`
  - `TCP Xmas scan / TCP fin scan / TCP null scan`
  - `UDP scan`
- [x] 上述每种扫描技术的实现测试均需要测试端口状态为：`开放`、`关闭` 和 `过滤` 状态时的程序执行结果
- [x] 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因
- [x] 在实验报告中详细说明实验网络环境拓扑、被测试`IP`的端口状态是如何模拟的
- [x] （可选）复刻 `nmap` 的上述扫描技术实现的命令行参数开关

## 实验步骤

### 0 主机状态设置

端口不是独立存在的，它是依附于进程的。某个进程开启，那么它对应的端口就开启了，进程关闭，则该端口也就关闭了。

* `Open`状态

  ```
  # 开启apache服务，以启用端口80
  service apache2 start
  
  #开启dnsmasq服务，以启用端口53
  apt-get install dnsmasq
  service dnsmasq start
  
  # 查看开启状态
  netstat -ntulp | grep 80 
  ```

  ![open80port](/chap0x05/img/open80port.png)

* `Close`状态

  ```
  # 关闭apache服务，以关闭端口80
  service apache2 stop
  
  # 查看80端口对应的进程
  lsof -i:80
  ```

  ![close80port](/chap0x05/img/close80port.png)
  
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

### 1 `TCP connect scan`

![tcpConnectScan](/chap0x05/img/tcpConnectScan.png)

#### 1.1 代码

* [tcpConnectScan.py](/chap0x05/code/tcpConnectScan.py)
* `nmap -sT -p80 ip`

#### 1.2 测试开放端口

* 扫描过程

  * `scapy`扫描

    ![tcpConnectScan-open-Scapy](/chap0x05/img/tcpConnectScan-open-Scapy.png)

  * `nmap` 扫描    (`nmap -p80 -sT 172.16.111.113`)

    ![tcpConnectScan-open-Nmap](/chap0x05/img/tcpConnectScan-open-Nmap.png)

* 数据包分析

  * [`tcpConnectScan-open.pcap`](/chap0x05/data/tcpConnectScan-open.pacp)

  * `scapy` 扫描
  
    ![image-20201023105013995](/chap0x05/img/tcs-o-s.png)
  
  * `nmap`扫描
  
    ![image-20201023105117239](/chap0x05/img/tcs-o-n.png)

#### 1.3 测试关闭端口

* 扫描过程

  * `scapy`扫描

    ![tcpConnectScan-closed-Scapy](/chap0x05/img/tcpConnectScan-closed-Scapy.png)

  * `nmap`扫描

    ![tcpConnectScan-closed-Nmap](/chap0x05/img/tcpConnectScan-closed-Nmap.png)

* 数据包分析

  * [`tcpConnectScan-closed.pcap`](/chap0x05/data/tcpConnectScan-closed.pcap)
  
  * `scapy`扫描
  
    ![image-20201023105343718](/chap0x05/img/tcs-c-s.png)
  
  * `nmap`扫描
  
    ![image-20201023105437099](/chap0x05/img/tcs-c-n.png)

#### 1.4 测试过滤端口

* 扫描过程

  * `Scapy`扫描

    ![tcpConnectScan-filtered-Nmap](/chap0x05/img/tcpConnectScan-filtered-Nmap.png)

  * `Nmap`扫描

    ![tcpConnectScan-filtered-Nmap](/chap0x05/img/tcpConnectScan-filtered-Nmap.png)

* 数据包分析

  * [`tcpConnectScan-filtered.pcap`](/chap0x05/data/tcpConnectScan-filtered.pcap)
  
  * `scapy`扫描
  
    ![image-20201023105654683](/chap0x05/img/tcs-f-s.png)
  
  * `nmap`扫描
  
    ![image-20201023105715693](/chap0x05/img/tcs-f-n.png)

### 2 `TCP stealth scan`

![tcpStealthScan](/chap0x05/img/tcpStealthScan.png)

#### 2.1 代码

* [tcpStealthScan.py](/chap0x05/code/tcpStealthScan.py)
* `nmap -sS -p80 ip`

#### 2.2 测试开放端口

* 扫描过程

  * `scapy`扫描

    ![tcpStealthScan-open-Scapy](/chap0x05/img/tcpStealthScan-open-Scapy.png)

  * `nmap`扫描

    ![tcpStealthScan-open-Nmap](/chap0x05/img/tcpStealthScan-open-Nmap.png)

* 数据包分析

  * [`tcpStealthScan-open.pcap`](/chap0x05/data/tcpStealthScan-open.pcap)

  * `scapy`扫描

    ![image-20201023110443508](/chap0x05/img/tss-o-s.png)

  * `nmap`扫描

    ![image-20201023110655324](/chap0x05/img/tss-o-n.png)

#### 2.3 测试关闭端口

* 扫描过程

  * `scapy`扫描

    ![tcpStealthScan-closed-Scapy](/chap0x05/img/tcpStealthScan-closed-Scapy.png)

  * `nmap`扫描

    ![tcpStealthScan-closed-nmap](/chap0x05/img/tcpStealthScan-closed-nmap.png)

* 数据包分析

  * [`tcpStealthScan-close.pcap`](/chap0x05/data/tcpStealthScan-close.pcap)
  
  * `scapy`扫描
  
    ![image-20201023111110085](/chap0x05/img/tss-c-s.png)
  
  * `nmap`扫描
  
    ![image-20201023111220633](/chap0x05/img/tss-c-n.png)

#### 2.4 测试过滤端口

* 扫描过程

  * `scapy`扫描

    ![tcpStealthScan-filtered-Scapy](/chap0x05/img/tcpStealthScan-filtered-Scapy.png)

  * `nmap`扫描

    ![tcpStealthScan-filtered-Nmap](/chap0x05/img/tcpStealthScan-filtered-Nmap.png)

* 数据包分析
  
  * [`tcpStealthScan-filtered.pcap`](/chap0x05/data/tcpStealthScan-filtered.pcap)
  
  * `scapy`扫描
  
    ![image-20201023111319489](/chap0x05/img/tss-f-s.png)
  
  * `nmap`扫描
  
    ![image-20201023111355946](/chap0x05/img/tss-f-n.png)

### 3 `TCP Xmas scan` 

  ![tcpXmasScan](/chap0x05/img/tcpXmasScan.png)

#### 3.1 代码

* [tcpXmasScan.py](/chap0x05/code/tcpXmasScan.py)
* `nmap -p80 -sX ip`

#### 3.2 测试开放端口

* 扫描过程

  * `scapy`扫描

    ![tcpXmasScan-open-Scapy](/chap0x05/img/tcpXmasScan-open-Scapy.png)

  * `nmap`扫描

    ![tcpXmasScan-open-Nmap](/chap0x05/img/tcpXmasScan-open-Nmap.png)

* 数据包分析

  * [tcpXmasScan-open.pcap](/chap0x05/data/tcpXmasScan-open.pcap)

  * `scapy`扫描

    ![image-20201023111510414](/chap0x05/img/txs-o-s.png)

  * `nmap`扫描

    ![image-20201023111549819](/chap0x05/img/txs-o-n.png)

#### 3.3 测试关闭端口

* 扫描过程

  * `scapy`扫描

    ![tcpXmasScan-closed-Scapy](/chap0x05/img/tcpXmasScan-closed-Scapy.png)

  * `nmap`扫描

    ![tcpXmasScan-closed-Nmap](/chap0x05/img/tcpXmasScan-closed-Nmap.png)

* 数据包分析

  * [tcpXmasScan-closed.pcap](/chap0x05/data/tcpXmasScan-closed.pcap)
  
  * `scapy`扫描
  
    ![txs-c-s](/chap0x05/img/txs-c-s.png)
  
  * `nmap`扫描
  
    ![txs-c-n](/chap0x05/img/txs-c-n.png)

#### 3.4 测试过滤端口

* 扫描过程

  * `scapy`扫描

    ![tcpXmasScan-filtered-Scapy](/chap0x05/img/tcpXmasScan-filtered-Scapy.png)

  * `nmap`扫描

    ![tcpXmasScan-filtered-Nmap](/chap0x05/img/tcpXmasScan-filtered-Nmap.png)

* 数据包分析

  * [tcpXmasScan-filtered.pcap](/chap0x05/data/tcpXmasScan-filtered.pcap)
  
  * `scapy`扫描
  
    ![txs-f-s](/chap0x05/img/txs-f-s.png)
  
  * `nmap`扫描
  
    ![txs-f-n](/chap0x05/img/txs-f-n.png)

### 4 `TCP fin scan`

![tcpFinScan](/chap0x05/img/tcpFinScan.png)

#### 4.1 代码

* [tcpFinScan.py](/chap0x05/code/tcpFinScan.py)
* `nmap -sF -p80 172.16.111.113`

#### 4.2 测试开放端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpFinScan-open-Scapy](/chap0x05/img/tcpFinScan-open-Scapy.png)
  
  * `nmap`扫描 
  
    ![tcpFinScan-open-Nmap](/chap0x05/img/tcpFinScan-open-Nmap.png)
* 数据包分析
  * [tcpFinScan-open.pcap](/chap0x05/data/tcpFinScan-open.pcap) 
  
  * `scapy`扫描
  
    ![tfs-o-s](/chap0x05/img/tfs-o-s.png)
  
  * `nmap`扫描
  
    ![tfs-o-n](/chap0x05/img/tfs-o-n.png)

#### 4.3 测试关闭端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpFinScan-closed-Scapy](/chap0x05/img/tcpFinScan-closed-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpFinScan-closed-Nmap](/chap0x05/img/tcpFinScan-closed-Nmap.png)
* 数据包分析
  * [tcpFinScan-closed.pcap](/chap0x05/data/tcpFinScan-closed.pcap)
  
  * `scapy`扫描
  
    ![tfs-c-s](/chap0x05/img/tfs-c-s.png)
  
  * `nmap`扫描
  
    ![tfs-c-n](/chap0x05/img/tfs-c-n.png)

#### 4.4 测试过滤端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpFinScan-filtered-Scapy](/chap0x05/img/tcpFinScan-filtered-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpFinScan-filtered-Nmap](/chap0x05/img/tcpFinScan-filtered-Nmap.png)
* 数据包分析
  * [tcpFinScan-filtered.pcap](/chap0x05/data/tcpFinScan-filtered.pcap)
  
  * `scapy`扫描
  
    ![tfs-f-s](/chap0x05/img/tfs-f-s.png)
  
  * `nmap`扫描
  
    ![tfs-f-n](/chap0x05/img/tfs-f-n.png)

### 5 `TCP null scan`

![tcpNullScan](/chap0x05/img/tcpNullScan.png)

#### 5.1 代码

* [tcpNullScan.py](/chap0x05/code/tcpNullScan.py)
* `nmap -sN ip`

#### 5.2 测试开放端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpNullScan-open-Scapy](/chap0x05/img/tcpNullScan-open-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpNullScan-open-Nmap](/chap0x05/img/tcpNullScan-open-Nmap.png)
* 数据包分析
  * [tcpNullScan-open.pcap](/chap0x05/data/tcpNullScan-open.pcap)
  
  * `scapy`扫描
  
    ![tns-o-s](/chap0x05/img/tns-o-s.png)
  
  * `nmap`扫描
  
    ![tns-o-n](/chap0x05/img/tns-o-n.png)

#### 5.3 测试关闭端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpNullScan-closed-Scapy](/chap0x05/img/tcpNullScan-closed-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpNullScan-closed-Nmap](/chap0x05/img/tcpNullScan-closed-Nmap.png)
* 数据包分析
  * [tcpNullScan-closed.pcap](/chap0x05/data/tcpNullScan-closed.pcap)
  
  * `scapy`扫描
  
    ![tns-c-s](/chap0x05/img/tns-c-s.png)
  
  * `nmap`扫描
  
    ![tns-c-n](/chap0x05/img/tns-c-n.png)

#### 5.4 测试过滤端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpNullScan-filtered-Scapy](/chap0x05/img/tcpNullScan-filtered-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpNullScan-filtered-Nmap](/chap0x05/img/tcpNullScan-filtered-Nmap.png)
* 数据包分析
  * [tcpNullScan-filtered.pcap](/chap0x05/data/tcpNullScan-filtered.pcap)
  
  * `scapy`扫描
  
    ![image-20201023114001960](/chap0x05/img/tns-f-s.png)
  
  * `nmap`扫描
  
    ![image-20201023114017097](/chap0x05/img/tns-f-n.png)

### 6 `UDP scan`

![udpScan](/chap0x05/img/udpScan.png)

#### 6.1 代码

* [udpScan.py](/chap0x05/code/udpScan.py)
* `nmap -sU -p53 ip`

#### 6.2 测试开放端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpNullScan-open-Scapy](/chap0x05/img/tcpNullScan-open-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpNullScan-open-Nmap](/chap0x05/img/tcpNullScan-open-Nmap.png)
* 数据包分析
  * [udpScan-open.pcap](chap0x05/data/udpScan-open.pcap)
  
  * `scapy`扫描
  
    ![us-o-s](/chap0x05/img/us-o-s.png)
  
  * `nmap`扫描
  
    ![us-o-n](/chap0x05/img/us-o-n.png)

#### 6.3 测试关闭端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpNullScan-closed-Scapy](/chap0x05/img/tcpNullScan-closed-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpNullScan-closed-Nmap](/chap0x05/img/tcpNullScan-closed-Nmap.png)
* 数据包分析
  * [udpScan-closed.pcap](chap0x05/data/udpScan-closed.pcap)
  
  * `scapy`扫描
  
    ![us-c-s](/chap0x05/img/us-c-s.png)
  
  * `nmap`扫描
  
    ![us-c-n](/chap0x05/img/us-c-n.png)

#### 6.4 测试过滤端口

* 扫描过程
  * `scapy`扫描
  
    ![tcpNullScan-filtered-Scapy](/chap0x05/img/tcpNullScan-filtered-Scapy.png)
  
  * `nmap`扫描
  
    ![tcpNullScan-filtered-Nmap](/chap0x05/img/tcpNullScan-filtered-Nmap.png)
* 数据包分析
  * [udpScan-filtered.pcap](chap0x05/data/udpScan-filtered.pcap)
  
  * `scapy`扫描
  
    ![us-f-s](/chap0x05/img/us-f-s.png)
  
  * `nmap`扫描
  
    ![us-f-n](/chap0x05/img/us-f-n.png)

## 实验总结

​		在自己搭建的内部网络中，分别用`Scapy`编程和`Nmap`的方式进行的端口扫描，其中`tcp connect scan`、`tcp stealth scan`、`tcp fin scan`、`tcp null scan`，使用两种方式都符合课本中的扫描方法原理，对于`udp scan`，在使用`Nmap`命令时，扫描主机发送的不是`UDP`数据包，而是`DNS`数据包。

## Q&A

* Q1：如何关闭端口？

  A：端口不是独立存在的，它是依附于进程的。某个进程开启，那么它对应的端口就开启了，进程关闭，则该端口也就关闭了。下次若某个进程再次开启，则相应的端口也再次开启。而不要纯粹的理解为关闭掉某个端口，不过可以禁用某个端口。

* Q2：如果先进行开放端口扫描，然后关闭进程再次扫描的结果有可能是`Flitered`，如果想得到`Closed`的扫描结果，可以重启目标主机。

## 参考资料

* [第五章网络扫描在线课件](https://c4pr1c3.github.io/cuc-ns-ppt/chap0x05.md.html)
* [NMAP 端口扫描](https://www.jianshu.com/p/909c7d72035c)
* [查看端口使用状态、关闭端口方法（netstat,lsof）](https://www.cnblogs.com/alantu2018/p/8462574.html)
* [如何用Scapy写一个端口扫描器](https://blog.csdn.net/think_ycx/article/details/50898096)
* [scapy官方文档](https://scapy.readthedocs.io/en/latest/)

## 课后问答题

1. 通过本章网络扫描基本原理的学习，试推测
   - 应用程序版本信息扫描原理
   - 网络漏洞扫描原理
2. 网络扫描知识库的构建方法有哪些？
3. 除了 `nmap` 之外，目前还有哪些流行的网络扫描器？和 `nmap` 进行优缺点对比分析

