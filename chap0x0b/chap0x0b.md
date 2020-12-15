# 常见蜜罐体验和探索



## 验目的

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

## PART1 ssh-Honeypot

### 1 蜜罐搭建

#### 安装`docker`环境

* 添加`docker-ce`的`apt`源

  ```shell
  apt-get update
  apt-get install -y apt-transport-https ca-certificates curl software-properties-common
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
  ```

* 开启`docker`服务并测试是否安装成功

  ```shell
  sudo systemctl start docker
  sudo docker run hello-world
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

### 1 蜜罐搭建

### 2 模仿攻击

## PART3 Canarytokens



## 实验总结

* 常见的蜜罐识别和检测方法

## Q&A

* `sudo apt-get install docker-ce`安装`docker-ce`的时候报错。

  解决：

  因为没找到合适的docker更新源，所以先添加docker源。并且需要在root用户下执行该条命令，`sudo`也不行。

  ```shell
  echo "deb https://download.docker.com/linux/ubuntu zesty edge" > /etc/apt/sources.list.d/docker.list
  ```

## 参考资料



## 课后问答题

