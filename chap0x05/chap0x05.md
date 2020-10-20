[toc]

# 基于Scapy编写端口扫描器

## 实验目的

- 掌握网络扫描之端口状态探测的基本原理

## 实验环境

- `python` + [`scapy`](https://scapy.net/)

- 局域网网络拓扑图

  ![NS-scan-network](/chap0x05/img/NS-scan-network.png)

## 实验要求

- 禁止探测互联网上的 `IP` ，严格遵守网络安全相关法律法规
- 完成以下扫描技术的编程实现
  - `TCP connect scan / TCP stealth scan`
  - `TCP Xmas scan / TCP fin scan / TCP null scan`
  - `UDP scan`
- 上述每种扫描技术的实现测试均需要测试端口状态为：`开放`、`关闭` 和 `过滤` 状态时的程序执行结果
- 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因
- 在实验报告中详细说明实验网络环境拓扑、被测试`IP`的端口状态是如何模拟的
- （可选）复刻 `nmap` 的上述扫描技术实现的命令行参数开关

## 实验步骤

### 1 `TCP connect scan`

#### 1.1 代码

[tcpConnectScan.py](/code/tcpConnectScan.py)

#### 1.2 测试开放端口



#### 1.3 测试关闭端口



#### 1.4 测试过滤端口



### 2 `TCP stealth scan`

### 3 `TCP Xmas scan` 

### 4 `TCP fin scan`

### 5 `TCP null scan`

### 6 `UDP scan`

## 实验总结





## Q&A





## 参考资料





## 补充：课后问答题

1. 通过本章网络扫描基本原理的学习，试推测
   - 应用程序版本信息扫描原理
   - 网络漏洞扫描原理

2. 网络扫描知识库的构建方法有哪些？

3. 除了 `nmap` 之外，目前还有哪些流行的网络扫描器？和 `nmap` 进行优缺点对比分析





