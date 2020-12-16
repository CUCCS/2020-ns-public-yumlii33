常见蜜罐体验和探索

[实验目的](#实验目的)

[实验环境](#实验环境)

[实验要求](#实验要求)

[实验步骤](#实验步骤)

* PART1 sshHoneypot

* PART2 Cowrie

* PART3 Canarytokens

[实验总结](#实验总结)

[Q&A](#Q&A)

[参考资料](#参考资料)

## 实验目的

- 了解蜜罐的分类和基本原理
- 了解不同类型蜜罐的适用场合
- 掌握常见蜜罐的搭建和使用

## 实验环境

- 从`paralax/awesome-honeypots`中选择 1 种低交互蜜罐和 1 种中等交互蜜罐进行搭建实验
  
  - 推荐 `SSH` 蜜罐
  
- 网络拓扑：

  ![](/chap0x0b/img/exp11网络拓扑.png)

## 实验要求

- [x] 记录蜜罐的详细搭建过程；
- [x] 使用 `nmap` 扫描搭建好的蜜罐并分析扫描结果，同时分析「 `nmap` 扫描期间」蜜罐上记录得到的信息；
- [x] 如何辨别当前目标是一个「蜜罐」？以自己搭建的蜜罐为例进行说明；
- [x] （可选）总结常见的蜜罐识别和检测方法；
- [x] （可选）基于 [canarytokens](https://github.com/thinkst/canarytokens) 搭建蜜信实验环境进行自由探索型实验；

## 实验步骤

在`Victim-Kali-exp11`中搭建蜜网，`Attacker-Kali-exp11`攻击`Victim-Kali-exp11`。

## PART1 sshHoneypot

```
这是是一种极低交互式的简易蜜罐。
```

### 1 蜜罐搭建

#### 安装`docker`环境

* 添加`docker-ce`的`apt`源

  ```shell
  sudo apt update
  sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
  ```

* 添加`docker`需要的密钥

  ```shell
  curl -fsSL https://download.daocloud.io/docker/linux/ubuntu/gpg | sudo apt-key add -
  ```

* 验证密钥的可用性

  ```shell
  sudo apt-key fingerprint 0EBFCD88
  ```

  ![image-20201215105516473](/chap0x0b/img/验证密钥的可用性.png)

* 安装`docker-ce`

  ```shell
  sudo apt-get install docker-ce
  
  #直接运行会报错，没找到合适的docker更新源，先添加docker源
  sudo echo "deb https://download.docker.com/linux/ubuntu zesty edge" > /etc/apt/sources.list.d/docker.list
  sudo apt update
  ```

* 开启`docker`服务并测试是否安装成功

  ```shell
  sudo systemctl start docker
  sudo docker run hello-world #可能报错，多执行几遍
   
  ```

  ![image-20201215112342254](/chap0x0b/img/开启docker服务并测试是否安装成功.png)

* 使用`docker image`指令查看镜像

  ![image-20201215112619845](/chap0x0b/img/dockerimages.png)

#### 安装`ssh-honeypot`

* 安装依赖包`libssh&libjson-c`

  ```shell
  apt install libssh-dev libjson-c-dev
  ```

* 配置`ssh`

  ```shell
  ssh-keygen -t rsa -f ./ssh-honeypot.rsa
  ```

  ![image-20201215112916251](/chap0x0b/img/配置ssh.png)

#### 安装`docker-ssh-honeypot`

* 安装

  ```shell
  git clone https://github.com/random-robbie/docker-ssh-honey
  
  docker build . -t local:ssh-honeypot
  ```

  ![image-20201215113459254](/chap0x0b/img/安装dockersshhoneypot库.png)

  ![image-20201215113732436](/chap0x0b/img/安装dshoneypot成功.png)

* 运行

  ```shell
  # 运行镜像 格式为本地端口:容器端口 
  sudo docker run -p 2234:22 local:ssh-honeypot
  ```

* 查看容器`id`，并进入容器

  ```shell
  sudo docker ps
  sudo docker exec -i -t container_id bash 
  ```

  ![image-20201215115010949](/chap0x0b/img/查看容器id并进入容器.png)

* 查看日志

  ```shell
  tail -F ssh-honeypot.log
  ```

  ![image-20201215115210965](/chap0x0b/img/测试查看日志功能.png)

### 2 模仿攻击

**`Attacker-Kali-exp11`对蜜罐所在主机端口进行`ssh`连接**

* ![image-20201215120444072](/chap0x0b/img/dockersshhoneypot攻击.png)

* 观察发现安装蜜罐的主机`Victim-Kali-exp11`日志记录了该行为。

* 刚开始输入命令`ssh molly@10.0.2.6`，显示和正常`ssh`登录一致，并且已经被记录到了日志。

* 然后尝试输入密码，但是总报错。换了`root`也不成功。原因是`ssh-honeypot`是一个低交互式的蜜罐，无法完成这些功能。但是用户能记录下攻击者的行为。

* 仔细查看日志信息，我们发现攻击者的所有行为都被记录下来了，包括输入的密码等等，以及攻击者的ip也展露无遗，这达到了蜜罐的初步目标，即收集对方的信息。

  ![image-20201215122505720](/chap0x0b/img/dockersshhoneypot日志记录了详细信息.png)

**`nmap` 扫描搭建好的蜜罐**

* 使用语句对搭建蜜罐的主机进行扫描。

  ![image-20201215123320317](/chap0x0b/img/nmap扫描dockersshhoneypot.png)

* 观察结果，发现用户主机并没有对nmap攻击进行记日志。说明该蜜罐并未对此生效，也再一次说明了该蜜罐是低交互式的简单蜜罐。

## PART2 Cowrie

```
Cowrie是一种中到高交互性的SSH和Telnet蜜罐。
```

### 1 蜜罐搭建

* 在`docker`中安装`Cowrie`

  ```shell
  sudo docker pull cowrie/cowrie
  ```

* 启动`cowrie`，使用端口`2222`

  ```shell
  sudo docker run -p 2222:2222 cowrie/cowrie
  ```

  ![image-20201215150724478](/chap0x0b/img/搭建cowrie环境.png)
  
* 查看日志

  ```shell
  sudo docker exec -i -t container_id bash #进入容器
  cat ~/cowrie-git/var/log/cowrie/cowrie.json #查看日志
  ```

### 2 模仿攻击

**`Attacker-Kali-exp11`对蜜罐所在主机端口进行`ssh`连接**

* 初次发起连接请求就有记录

  ![image-20201216090515851](/chap0x0b/img/cowrie蜜罐初次连接.png)

* 使用普通用户登录不成功，且三次不成功后自动结束，符合正常密码输错的情况。使用`root`用户登录成功，进`shell`界面。

* ![image-20201216091148161](/chap0x0b/img/cowrie蜜罐root登录成功.png)

* 长时间不适用会自动退出ssh连接。二次连接直接输入密码，不用输入`yes`，也和正常的`ssh`一致。测试用不正确的密码连接，看起来也成功登录了！体现的蜜罐思想。

  ![image-20201216091954778](/chap0x0b/img/cowrie蜜罐不正确的密码也登录成功了.png)

* 如果`Vicitm`主机的蜜罐环境重启，连接时会显示更新`host key`

  ![image-20201216100055828](/chap0x0b/img/cowrie蜜罐重启后再次连接.png)

* 查看日志文件。和前面不同，这不能再容器里实时显示，会在本次连接结束后以`json`的格式保存日志，包含了用户的操作过程和基本信息。

  ![image-20201216100354894](/chap0x0b/img/cowrie蜜罐日志.png)

**在蜜罐系统中执行常用操作**

* 切换用户操作`su molly`

  ![image-20201216101159603](/chap0x0b/img/cowrie蜜罐切换用户命令无反应.png)

* 网络连通性测试操作`ping www.baidu.com`、`curl https://www.baidu.com`

  ![image-20201216101800334](/chap0x0b/img/cowrie蜜罐网络连通性测试.png)

* 忘记停止`ping`命令就去写报告，结果还是自动断开连接了，但是在`Victim`主机里，命令行一直在输出`Connection was probably lost. Could not write to terminal`重新连接后也继续写，可能是没有识别出来该会话已经断开。需要重新启动才行。

  ![image-20201216101954848](/chap0x0b/img/cowrie蜜罐一直输出有点问题.png)

* `curl`输入错误的`url`，发现不会返回错误信息，并且不能手动终止，只能等待超时退出。

  ![image-20201216103025868](/chap0x0b/img/cowrie蜜罐curl没有报错.png)

* 安装包操作`apt-get install xxx`和`apt-get update`

  ![image-20201216104443131](/chap0x0b/img/cowrie蜜罐安装包.png)

**nmap扫描**

发现`nmap`端口扫描不会被记录。

![image-20201216105752833](/chap0x0b/img/cowrie蜜罐nmap扫描.png)

## PART3 Canarytokens

```
honeytokens是可以“以快速的，便捷的方式帮助防御方发现他们已经被攻击了的客观事实”。为了实现这个目标，我们可以使用Canarytokens应用来生成token,当入侵者访问或者使用由Canarytokens应用生成的honeytoken时，该工具将会通过邮件通知我们，并附带异常事件的细节说明。
```

### 3.1 环境配置

* 克隆仓库并进入目录

  ```
  git clone https://github.com/thinkst/canarytokens-docker
  cd canarytokens-docker
  ```

* 安装`docker-compose`

  ```
  sudo apt update
  sudo apt-get install libyaml-dev # 安装依赖包
  sudo apt-get install python3-pip python-dev 
  sudo pip install -U docker-compose
  ```

* 修改配置文件

  `frontend.env`

  ```shell
  CANARY_DOMAINS=proshare.net 
  CANARY_NXDOMAINS=proshare.net
  ```

  `switchboard.env`

  ![image-20201216113859685](/chap0x0b/img/switch配置修改.png)

  ```shell
  CANARY_MAILGUN_DOMAIN_NAME=MOLLYY
  CANARY_MAILGUN_API_KEY=key123123
  #CANARY_MANDRILL_API_KEY=
  #CANARY_SENDGRID_API_KEY=
  CANARY_PUBLIC_IP=162.243.117.221
  CANARY_PUBLIC_DOMAIN=proshare.net
  CANARY_ALERT_EMAIL_FROM_ADDRESS=noreply@proshare.net
  CANARY_ALERT_EMAIL_FROM_DISPLAY=Canarytoken
  CANARY_EMAIL_SUBJECT=Canarytoken
  ```

* 启动

  ```shell
  sudo docker-compose up
  ```

  ![image-20201217074118741](/chap0x0b/img/启动成功.png)

* 正常应该能通过IP地址访问，但是实际操作访问不通，时间原因没有继续做。

  ![image-20201217074715792](/chap0x0b/img/无法通过ip地址访问.png)

## 实验总结

* 常见的蜜罐识别和检测方法

  * `ssh`连接的时候，测试不同密码和用户，有时输入什么密码都可以登录成功，而且登录时的提示信息有不同，可以感觉出来不是正常的系统。
  * 执行一些常见命令的时候，比如在`root`用户下执行`apt update`竟然提示权限错误，显然是不正常的，安装已安装的包不会提示已存在，安装错误的包不会提示找不到源，这也能判断出错误。
  * 通过查看一些特殊文件，特殊路径，如果没有应有的文件，也是不正常的。
  * 课堂上老师提到，可以通过检查环境信息来分辨。因为很容易就能知道进行物理机器应有的配置，版本号等信息，而搭建的蜜罐往往在这方面做的不够完善，容易露陷。
  * 网上查到到，用`Nmap`等`Scan`工具，同一个机器同时开放很多Port的。
  * 如果是`VMware`虚拟机，重点关注MAC地址的范围。

  

## Q&A

* `sudo apt-get install docker-ce`安装`docker-ce`的时候报错。

  解决：

  因为没找到合适的docker更新源，所以先添加docker源。并且需要在root用户下执行该条命令，`sudo`也不行。

  ```shell
  echo "deb https://download.docker.com/linux/ubuntu zesty edge" > /etc/apt/sources.list.d/docker.list
  ```
  
* 修改配置文件的时候，报错：

  ![image-20201216114312713](/chap0x0b/img/配置文件报错.png)

  解决：

  修改`frontend.env.dist`的文件名为`frontend.env`，`switchboard.env.dist`的文件名为`switchboard.env`。

  ```shell
  sudo mv frontend.env.dist frontend.env
  sudo mv switchboard.env.dist switchboard.env
  ```
  
  成功。
  
* docker-compose up 报错

  ![image-20201216184431355](/chap0x0b/img/空间不够报错.png)

  解决：
  
  查看磁盘空间
  
  ![image-20201216215540005](/chap0x0b/img/磁盘空间.png)
  
  磁盘空间空间不足，无法安装。
  
  网上搜索可以扩展磁盘空间，但是由于之前用的是多重加载建立的`Victim`主机，操作怕不成功，而且多重加载的镜像重新建立一个虚拟机非常快，所以选择重建。
  
  （忘了备份docker环境，还要重装docker...）

## 参考资料

* [droberson/ssh-honeypot](https://github.com/droberson/ssh-honeypot)
* [cowrie/cowrie](https://github.com/cowrie/cowrie)
* [random-robbie/docker-ssh-honey](https://github.com/random-robbie/docker-ssh-honey)
* [如何用Canarytokens搭建蜜罐并检测可疑入侵](https://blog.csdn.net/qq_40907977/article/details/106101899)
* [第十一章课件](https://c4pr1c3.github.io/cuc-ns-ppt/chap0x11.md.html#/title-slide)
* [解决Virtualbox中no space left on device问题](https://www.leeguangxing.cn/blog_post_82.html)



