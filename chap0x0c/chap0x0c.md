# 计算机取证：使用 zeek 来完成取证分析



## 实验目的

* 模拟计算机取证

## 实验要求

* 使用 zeek 来完成取证分析

## 实验环境

* `Kali Rolling`

## 实验步骤

<h3 id="1">1 下载zeek</h3>

* 安装依赖包

  ```bash
  sudo apt update
  sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python-dev swig zlib1g-dev
  ```

* 下载`zeek`源码包（使用校园网校内`ftp`访问`url`下载

  ```bash
  wget http://sec.cuc.edu.cn/ftp/soft/zeek-3.0.0.tar.gz
  ```

* 解压缩`zeek`源码

  ```bash
  tar zxf zeek-3.0.0.tar.gz
  ```

* 进入`zeek`源码解压缩后目录

  ```bash
  cd zeek-3.0.0
  ```

* 生成构建脚本

  ```bash
  ./configure
  ```

* 构建成功后安装到构建脚本默认指定路径`/usr/local/zeek`

  ```bash
  make && make install
  ```

* 将` zeek` 可执行文件目录添加到当前用户的` PATH `环境变量

  ```bash
  if [[ $(grep -c '/usr/local/zeek/bin' ~/.bashrc) -eq 0 ]];then echo 'export PATH=/usr/local/zeek/bin:$PATH' >> ~/.bashrc;fi
  ```

* 重新读取 `~/.bashrc` 以使环境变量设置即时生效

  ```bash
  source ~/.bashrc
  ```

<h3 id="2">2 实验环境基本信息</h3>

<h3 id="3">3 编辑zeek配置文件</h3>

<h3 id="4">4 使用zeek自动化分析pacp文件</h3>

## 实验总结

## Q&A

## 参考资料



