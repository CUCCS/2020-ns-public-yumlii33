# 基于VirtualBox的网络攻防基础环境搭建

## 实验目的

- 掌握 VirtualBox 虚拟机的安装与使用；
- 掌握 VirtualBox 的虚拟网络类型和按需配置；
- 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境

- VirtualBox 虚拟机
- 攻击者主机（Attacker）：Kali Rolling 2109.2
- 网关（Gateway, GW）：Debian Buster
- 靶机（Victim）：From Sqli to shell / xp-sp3 / Kali

## 实验要求

- 虚拟硬盘配置成多重加载，效果如下图所示；

  ![img](/chap0x01/img/vb-multi-attach.png)

- 搭建满足如下拓扑图所示的虚拟机网络拓扑；

  ![img](/chap0x01/img/vb-exp-layout.png)

> 根据实验宿主机的性能条件，可以适度精简靶机数量

- 完成以下网络连通性测试；
  - [x] 靶机可以直接访问攻击者主机
  - [x] 攻击者主机无法直接访问靶机
  - [x] 网关可以直接访问攻击者主机和靶机
  - [x] 靶机的所有对外上下行流量必须经过网关
  - [x] 所有节点均可以访问互联网

## 实验步骤

### 1、创建安装虚拟机

基本步骤和上学期《linux系统与网络管理》课程第一次安装`ubuntu18.04`的步骤一致，一些差异和要注意的点在下面列出。

* 安装`windows xp-sp3`

  * 创建虚拟介质的时候选择`vmdk`，否则可能创建不成功。
* 安装`Debian Buster`

  * 在安装过程中选择「要安装的软件」时，不需要选择任何图形化界面的包，只需要安装一个 `OpenSSH Server` 和 `standard system utilities`。

  * 安装过程选择「国内源」，否则下载软件的速度很慢。
* 安装`Kali`
  * 新建虚拟机选择版本时没有`kali`选项，选择`Debian`。

### 2、虚拟硬盘配置成多重加载并创建拓扑图里用到的机器

* 虚拟介质设置成「多重加载」

  ![image-20200922095552479](/chap0x01/img/多重加载虚拟硬盘.png)

* 创建多台机器，选择「已有的虚拟硬盘」，从多重加载的虚拟硬盘创建

  ![image-20200922095851653](/chap0x01/img/拓扑图机器.png)

  ![image-20200922100206746](/chap0x01/img/多台主机详细设置.png)

### 3、配置网关主机

#### 3.1配置网关主机的网卡

* `NET`网络
* `Host-Only`网络
* 内部网络（`intnet1`)
* 内部网络（`intnet2`)

![image-20200930094529828](/chap0x01/img/网关网络设置.png)

#### 3.2配置网关转发规则

* 修改`/etc/network/interfaces`（先备份），使用命令`/etc/init.d/networking restart`重新启动网络（或者使用`/sbin/ifdown `和`/sbin/ifup `）。先启用enp0s8，方便使用`ssh`连接网关主机。

* 修改`etc/network/interfaces`配置文件，重新启动网卡（`enp0s3`, `enp0s8`, `enp0s9`, `enp0s10`）

  ![image-20200929115353311](/chap0x01/img/etc-network-interfaces.png)

* 配置好的网关网络详细信息如下

  ![image-20200929115248239](/chap0x01/img/网关ip.png)

#### 3.3配置`dnsmasq`

* 配置`dnsmasq`可以实现自动获取`IP`地址，可以实现域名解析。

* 安装`dnsmasq`：`apt-get install dnsmasq`

* 添加配置文件

  ```
  # /etc/dnsmasq.d/gw-enp010.conf
  interface=enp0s10
  dhcp-range=172.16.222.100,172.16.222.150,240h
  ```

  ```
  # /etc/dnsmasq.d/gw-enp09.conf
  interface=enp0s9
  dhcp-range=172.16.111.100,172.16.111.150,240h
  ```

* 修改主配置文件（修改前先备份原文件）

  ```
  # /etc/dnsmasq.conf
  # diff dnsmasq.conf dnsmasq.conf.bak
  
  661,662c661
  < log-queries
  < log-facility=/var/log/dnsmasq.log
  ---
  > #log-queries
  665c664
  < log-dhcp
  ---
  > #log-dhcp
  ```

### 4、配置`intnet1`里的靶机

#### 4.1`Victim-XP-1`

* 设置网络->`内部网络`->`intnet1` （控制芯片没有更改也可以在虚拟机里看到网卡，所以没有修改控制芯片）

  ![image-20200930095317562](/chap0x01/img/xp-1-网络设置.png)

* 因为前面已经设置了`dnsmasq`，所以自动获取正确的`IP`地址

  ![image-20200930150822764](/chap0x01/img/ip-xp-1.png)

#### 4.2`Victim-Kali-1`

* 设置网络->`内部网络`->`intnet1`

* `IP`地址

  ![image-20200930152321054](/chap0x01/img/ip-kali-1.png)

### 5、配置`intnet2`里的靶机

#### 5.1`Victim-XP-2`

* 设置网络->`内部网络`->`intnet2`

* `IP`地址

  ![image-20200930151844727](/chap0x01/img/ip-xp-2.png)

#### 5.2`Victim-DEbian-2`

* 设置网络->`内部网络`->`intnet2`

* `IP`地址

  ![image-20200930150932230](/chap0x01/img/ip-debian-2.png)

### 6. 配置`Attacker-Kali`攻击者主机

* 网络选择`net network`，名称和网关的`net network`网络的名称一致。

  ![image-20200930151615790](/chap0x01/img/Attacker-Kali-网卡设置.png)

* IP地址

  ![image-20200930152426767](/chap0x01/img/ip-kali-attacker.png)

### 7. 连通性测试

连通性测试使用如下四台虚拟机进行。

![image-20200930141414783](/chap0x01/img/连通性测试-四台代表机器.png)

#### 7.1靶机可以直接访问攻击者主机

![image-20200930141608792](/chap0x01/img/连通性测试-靶机访问攻击者主机.png)

#### 7.2攻击者主机无法直接访问靶机

![image-20200930141835832](/chap0x01/img/连通性测试-攻击者主机无法直接访问靶机.png)

#### 7.3网关可以直接访问攻击者主机和靶机

![image-20200930142055009](/chap0x01/img/连通性测试-网关可以直接访问攻击者主机和靶机.png)

#### 7.4靶机的所有对外上下行流量必须经过网关

1. 预装`tmux`和`tcpdump`，方便后续查看记录

2. `tcpdump -i enp0s10 -n -w 20200930.1.pcap`：将输出存入数据包`20200930.xp.1.pcap`

   ![image-20200930144903327](/chap0x01/img/连通性测试-将记录输入数据包.png)

3. 将数据包拷贝到`windows`主机，使用`Wireshark` 查看分析

   `scp molly@192.168.56.106:/home/molly/20200930.xp.1.pcap  ./`

#### 7.5所有节点均可以访问互联网

![image-20200930142320335](/chap0x01/img/连通性测试-所有节点均可以访问互联网.png)

## Q&A

* Q：`ipconfig /renew` 失败

  A：http://www.imooc.com/wenda/detail/512074 ，没关固定分配。

* Q：`Attacker`和`GW`的`net网络`都是`10.0.2.15`

  ![image-20200930004926637](/chap0x01/img/NAT模式ip一样.png)

  A：使用命令` /sbin/ifdown enp0s3 && /sbin/ifup enp0s3`后解决：

  ![image-20200930004822274](/chap0x01/img/更新enp0s3的ip.png)

* Q：靶机和`Attacker`都可以`ping`通网关的`Host-Only`网络的`IP`地址，老师课上实验不能`ping`通。

  A：理解错误。能`ping`通网关主机，但是`ping`不通`Host-Only`网络。

  ![image-20201001143439195](/img/pingHostOnly.png)

* Q：`Attacker`可以`ping`通`GW`？

* A：可以。

## 参考资料

* [c4pr1c3/diff dnsmasq.conf dnsmasq.conf.bak](https://gist.github.com/c4pr1c3/8d1a4550aa550fabcbfb33fad9718db1)
* [ 基于 VirtualBox 的网络攻防基础环境搭建](https://c4pr1c3.github.io/cuc-ns/chap0x01/exp.html)

## 补充：课后问答题

以下⾏为分别破坏了CIA和AAA中哪⼀个属性或多个属性？

- 小明抄小强的作业
  - 「信息泄露」，破坏了机密性（Confidentiality）。
- 小明把小强的系统折腾死机了
  - 小强的电脑「拒绝服务」，破坏了可用性（Availability）。
- 小明修改了小强的淘宝订单
  - 信息被「篡改」，破坏了完整性（Integrity）。
- 小明冒充小强的信用卡账单签名
  - 信息被「篡改」和伪造签名导致身份被「假冒」，破坏了完整性（Integrity），认证性（Authentication）。

- 小明把自⼰电脑的IP修改为小强电脑的IP，导致小强的电脑⽆法上⽹
  - 小明「假冒」小强的身份，同时小强的电脑「拒绝服务」，破坏了认证性Authentication），可用性（Availability）。