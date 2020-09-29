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

  ![img](\img\vb-multi-attach.png)

- 搭建满足如下拓扑图所示的虚拟机网络拓扑；

![img](\img\vb-exp-layout.png)

> 根据实验宿主机的性能条件，可以适度精简靶机数量

- 完成以下网络连通性测试；
  - [ ] 靶机可以直接访问攻击者主机
  - [ ] 攻击者主机无法直接访问靶机
  - [ ] 网关可以直接访问攻击者主机和靶机
  - [ ] 靶机的所有对外上下行流量必须经过网关
  - [ ] 所有节点均可以访问互联网

## 实验步骤

### 1. 创建安装虚拟机

* 安装`windows xp-sp3`

  * 创建虚拟介质的时候选择`vmdk`，否则可能创建不成功。

* 安装`Debian Buster`

  * 在安装过程中选择「要安装的软件」时，不需要选择任何图形化界面的包，只需要安装一个 OpenSSH Server 和 standard system utilities。

  * 安装过程选择国内源，否则下载软件的速度很慢。

* 安装`Kali`

### 2. 虚拟硬盘配置成多重加载并创建拓扑图里用到的机器

* 设置多重加载

  ![image-20200922095552479](\img\多重加载虚拟硬盘.png)

* 创建多台机器，选择`已有的虚拟硬盘`，从多重加载的虚拟硬盘创建。

  ![image-20200922095851653](\img\拓扑图机器.png)

  ![image-20200922100206746](\img\多台主机详细设置.png)

### 3. 配置网关

* 配置网关主机的网络

  ![image-20200929103503660](img\网关网络设置.png)

* 修改`/etc/network/interfaces`，使用命令`/etc/init.d/networking restart`重新启动网络（或者`/sbin/ifdown `和`/sbin/ifup `）先启用enp0s8，方便使用`ssh`连接网关主机

* 粘贴`etc/network/interfaces`配置文件，重新启动网卡

  ![image-20200929115353311](\img\etc-network-interfaces.png)

* 配置好的网关`ip`

  ![image-20200929115248239](\img\网关ip.png)

### 4、配置`intnet1`



## 参考资料

* [c4pr1c3/diff dnsmasq.conf dnsmasq.conf.bak](https://gist.github.com/c4pr1c3/8d1a4550aa550fabcbfb33fad9718db1)