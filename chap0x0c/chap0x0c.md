

# 计算机取证：使用 zeek 来完成取证分析

[实验目的](#实验目的)

[实验要求](#实验要求)

[实验环境](#实验环境)

[实验步骤](#实验步骤)

* [1 下载zeek](#1)

* [2 实验环境基本信息](#2)

* [3 编辑zeek配置文件](#3)

* [4 使用zeek自动化分析pacp文件](#4)

* [5 Zeek 的一些其他技巧](#5)

[实验总结](#实验总结)

[Q&A](#Q&A)

[参考资料](#参考资料)

## 实验目的

* 模拟计算机取证

## 实验要求

- [x] 使用 zeek 来完成取证分析

## 实验环境

* `Kali Rolling 2020.3`
* `Zeek 4.1.0-dev.10`

## 实验步骤

<h3 id="1">1 下载zeek</h3>

* 安装依赖包

  ```bash
  sudo apt update
  sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python-dev swig zlib1g-dev
  ```

* 下载`zeek`源码包（官网下载）

  ```bash
  # wget http://sec.cuc.edu.cn/ftp/soft/zeek-3.0.0.tar.gz
  
  git clone --recursive https://github.com/zeek/zeek
  ```

* 进入`zeek`源码目录

  ```bash
  cd zeek
  ```

* 生成构建脚本

  ```bash
  ./configure
  ```

* 构建成功后安装到构建脚本默认指定路径`/usr/local/zeek`

  ```bash
  sudo make && make install
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

```bash
cat /etc/os-release
# PRETTY_NAME="Kali GNU/Linux Rolling"
# NAME="Kali GNU/Linux"
# ID=kali
# VERSION="2020.3"
# VERSION_ID="2020.3"
# VERSION_CODENAME="kali-rolling"
# ID_LIKE=debian
# ANSI_COLOR="1;31"
# HOME_URL="https://www.kali.org/"
# SUPPORT_URL="https://forums.kali.org/"
# BUG_REPORT_URL="https://bugs.kali.org/"

uname -a
# Linux bogon 5.7.0-kali1-amd64 #1 SMP Debian 5.7.6-1kali2 (2020-07-01) x86_64 GNU/Linux

zeek -v
# zeek version 4.1.0-dev.10
```

![image-20201220063351858](/chap0x0c/img/实验环境基本信息.png)

<h3 id="3">3 编辑zeek配置文件</h3>

- 编辑 `/usr/local/zeek/share/zeek/site/local.zeek` ，在文件尾部追加两行新配置代码

```bash
@load frameworks/files/extract-all-files
@load mytuning.zeek
```

![](/chap0x0c/img/local.zeek配置.png)

- 在 `/usr/local/zeek/share/zeek/site` 目录下创建新文件 `mytuning.zeek` ，[内容为](https://www.bro.org/documentation/faq.html#why-isn-t-bro-producing-the-logs-i-expect-a-note-about-checksums)：

```bash
redef ignore_checksums = T;
```

![](/chap0x0c/img/mytuning配置.png)

<h3 id="4">4 使用zeek自动化分析pacp文件</h3>

* 下载实验说明中提供的`pcap`包文件

  ```bash
  sudo wget https://c4pr1c3.github.io/cuc-ns/chap0x12/attack-trace.pcap
  ```

  ![image-20201220064753576](/chap0x0c/img/下载pcap包.png)

* 使用`zeek`自动化分析`pcap`文件

  ```bash
  zeek -r attack-trace.pcap /usr/local/zeek/share/zeek/site/local.zeek
  ```

* 出现了警告信息`WARNING: No Site::local_nets have been defined. It's usually a good idea to define your local networks.`，对于本次入侵取证实验来说没有影响。不过进行编辑 `mytuning.zeek`即可解决，由于报的警告是变量未定义，于是增加一行变量定义。需要注意的是添加和不添加上述一行变量定义除了 `zeek` 运行过程中是否会产生警告信息的差异，增加这行关于本地网络 `IP` 地址范围的定义对于本次实验来说会新增 2 个日志文件，会报告在当前流量（数据包文件）中发现了本地网络`IP`和该`IP`关联的已知服务信息。

  ![image-20201220065944359](/chap0x0c/img/mytuning增加一行.png)

  再次执行，不报错：

  ![image-20201220070147927](/chap0x0c/img/再次执行不报错.png)

* 在 `attack-trace.pcap` 文件的当前目录下会生成一些 `.log` 文件和一个 `extract_files` 目录，在该目录下我们会发现有一个文件。

  ```bash
  file extract-1240198114.648099-FTP_DATA-FutkFK231QTWleGBs9
  extract-1240198114.648099-FTP_DATA-FutkFK231QTWleGBs9: PE32 executable (GUI) Intel 80386, for MS Windows
  ```

  ![image-20201220071031202](/chap0x0c/img/file.png)

* 将该文件上传到 [virustotal](https://virustotal.com/) 我们会发现匹配了一个 [历史扫描报告](https://virustotal.com/en/file/b14ccb3786af7553f7c251623499a7fe67974dde69d3dffd65733871cddf6b6d/analysis/) ，该报告表明这是一个已知的后门程序！至此，基于这个发现就可以进行逆向倒推，寻找入侵线索了。

  ![image-20201220074111910](/chap0x0c/img/上传网站.png)

* 阅读 `/usr/local/zeek/share/zeek/base/files/extract/main.zeek` 的源代码。了解到该文件名的最右一个-右侧对应的字符串 `FHUsSu3rWdP07eRE4l` 是 `files.log` 中的文件唯一标识。

  ```bash
  function on_add(f: fa_file, args: Files::AnalyzerArgs)
          {
          if ( ! args?$extract_filename )
                  args$extract_filename = cat("extract-", f$last_active, "-", f$source,
                                              "-", f$id);
  
          f$info$extracted = args$extract_filename;
          args$extract_filename = build_path_compressed(prefix, args$extract_filename);
          f$info$extracted_cutoff = F;
          mkdir(prefix);
          }
  ```

* 通过查看 `files.log` ，发现该文件提取自网络会话标识（ `zeek` 根据 IP 五元组计算出的一个会话唯一性散列值）为 `Ca4P5U2d542zW0mxE5` 的 FTP 会话。

  ![image-20201220071559713](/chap0x0c/img/唯一标识.png)

* 可以确定这是用户通过 ftp 上传的恶意的可执行文件，根据文件标识 (files.log) 找到会话标识 (conn.log)，进而找到该恶意用户的 ip 地址,该 `Ca4P5U2d542zW0mxE5` 会话标识在 `conn.log` 中可以找到对应的 IP 五元组信息。

  ```bash
   cat files.log | zeek-cut conn_uids
  ```

  ![image-20201220074357599](/chap0x0c/img/会话标识.png)

* 通过 `conn.log` 的会话标识匹配，我们发现该PE文件来自于IPv4地址为：`98.114.205.102` 的主机。

  ![image-20201220074629632](/chap0x0c/img/找到ip.png)

<h3 id="5">5 Zeek 的一些其他技巧</h3>

* `ftp.log` 中默认不会显示捕获的 FTP 登录口令，我们可以通过在 `/usr/local/zeek/share/zeek/site/mytuning.zeek` 中增加以下变量重定义来实现

  ```bash
  redef FTP::default_capture_password = T;
  ```

  ![image-20201220074805872](/chap0x0c/img/重定义实现.png)

  修改后的配置：

  ![image-20201220075000572](/chap0x0c/img/ftp.log.png)

* 使用正确的分隔符进行过滤显示，提高可读性

  ```bash
  # 从头开始查看日志文件，显示前1行
  head -n1 conn.log
  
  # Bro的日志文件默认使用的分隔符显示为ASCII码\x09，通过以下命令可以查看该ASCII码对应的“可打印字符”
  echo -n -e '\x09' | hexdump -c
  
  # 使用awk打印给定日志文件的第N列数据
  awk -F '\t' '{print $3}' conn.log
  ```

  ![image-20201220075330797](/chap0x0c/img/提高可读性.png)

- 查看Bro的超长行日志时的横向滚动技巧

  ```bash
  less -S conn.log
  ```

  ![image-20201220075516515](/chap0x0c/img/超长日志行查看.png)

- 使用 `zeek-cut` 更“优雅”的查看日志中关注的数据列

  ```bash
  # 查看conn.log中所有可用的“列名”
  grep ^#fields conn.log | tr '\t' '\n'
  
  # 按照“列名”输出conn.log中我们关注的一些“列”
  zeek-cut ts id.orig_h id.orig_p id.resp_h id_resp_p proto < conn.log
  
  # 将UNIX时间戳格式转换成人类可读的时间（但该方法对于大日志文件处理性能非常低）
  zeek-cut -d < conn.log
  ```

  ![](/chap0x0c/img/查看数据列.png)

## 实验总结

`Zeek`是一款被动的开源网络流量分析器，它的主要用作一种**安全监视器**，可以对链接上的所有流量进行深入检查，并且生成大量的日志文件，以便于查找可疑活动的迹象。

## Q&A

* 从`sec.cuc.edu.cn`下载不成功

  ![image-20201219181611165](/chap0x0c/img/wgetsec.cuc.edu.cn无响应.png)

  解决：

  访问官网下载。

  ![image-20201219181719978](/chap0x0c/img/无法访问此网站.png)

* 自动化分析命令报错：`permission denied`

  解决：

  使用`sudo`也无法解决，最终切换`root`用户，解决。

  ![image-20201220065602781](/chap0x0c/img/切换root用户.png)

## 参考资料

* [Zeek 官方安装指南](https://docs.zeek.org/en/stable/install/install.html) 
* [第十二章 实验](https://c4pr1c3.github.io/cuc-ns/chap0x12/exp.html)

