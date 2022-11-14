---
date: 2022-06-30-星期四 09:41:53
update: 
tags: [time/year2022,time/month06]
id: 20220630094153
---

# John使用手册

[blog.csdn.net](https://blog.csdn.net/i_can1/article/details/107227565?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165655324816781435457364%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165655324816781435457364&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-107227565-null-null.142^v26^control,157^v15^new_3&utm_term=john&spm=1018.2226.3001.4187)

### 一、什么是 John the Ripper ？

看到这个标题，想必大家都很好奇，`John the Ripper` 是个什么东西呢？如果直译其名字的话就是： `John` 的撕裂者(工具)。 相比大家都会觉得摸不着头脑，撕裂者是啥玩意啊？

事实上，`John the Ripper` 是一款大受欢迎的、免费的开源软件。也是一个基于字典的快速破解密码的工具，是一款用于在已知密文的情况下尝试破解出明文的破解密码软件，支持目前大多数的加密算法，如 `DES` 、 `MD4` 、 `MD5` 等。 `John the Ripper` 支持字典破解方式和暴力破解方式。它支持多种不同类型的系统架构，包括 `Unix` 、 `Linux` 、 `Windows` 、 `DOS` 模式、 `BeOS` 和 `OpenVMS` ，主要目的是破解不够牢固的 `Unix/Linux` 系统密码。

如果你想使用专门针对特定操作系统优化、并生成相应本地代码的商业版本的该产品，那么你可以使用 `John the Ripper Pro` ，主页地址在这里：[http://www.openwall.com/john/pro/，你可以根据你的需要去购买或者下载破解字典表。本文以](http://www.openwall.com/john/pro/%EF%BC%8C%E4%BD%A0%E5%8F%AF%E4%BB%A5%E6%A0%B9%E6%8D%AE%E4%BD%A0%E7%9A%84%E9%9C%80%E8%A6%81%E5%8E%BB%E8%B4%AD%E4%B9%B0%E6%88%96%E8%80%85%E4%B8%8B%E8%BD%BD%E7%A0%B4%E8%A7%A3%E5%AD%97%E5%85%B8%E8%A1%A8%E3%80%82%E6%9C%AC%E6%96%87%E4%BB%A5) `Linux` 下的 `John the Ripper` 为例来给大家讲解其用法。

目前 `John the Ripper` 的最新版本是 `1.8.0` ，我们可以通过输入 `john` 命令去查看当前的版本信息和帮助信息。

![](https://ask.qcloudimg.com/http-save/yehe-1564476/8ow10t00q8.png?imageView2/2/w/1620)

如果想了解 `John the Ripper` 的最新动态，请参看官网：[https://www.openwall.com/john/](https://www.openwall.com/john/)

如果你想添加新功能， `Github` 项目链接在这里：[https://github.com/magnumripper/JohnTheRipper](https://github.com/magnumripper/JohnTheRipper)

### 二、John the Ripper 的安装和使用

`John the Ripper` 在 `Windows` 、 `Linux` 和 `MacOS` 都有对应的安装包，去官网下载即可。

官网链接：[https://www.openwall.com/john/](https://www.openwall.com/john/)

而在这里我就介绍 `Debian/Ubuntu` 的简便安装方法。由于在 `apt` 仓库中已经内置了 `John the Ripper` ，我们只需要通过以下命令即可安装。

    sudo apt-get install john
           

![](https://ask.qcloudimg.com/http-save/yehe-1564476/icnh8d4k0l.png?imageView2/2/w/1620)

`john` 命令的具体参数选项如下表：

选 项

描 述

\--single

single crack 模式，使用配置文件中的规则进行破解

\--wordlist=FILE--stdin

字典模式，从 FILE 或标准输入中读取词汇

\--rules

打开字典模式的词汇表切分规则

\--incremental\[=MODE\]

使用增量模式

\--external=MODE

打开外部模式或单词过滤，使用 \[List.External:MODE\] 节中定义的外部函数

\--stdout\[=LENGTH\]

不进行破解，仅仅把生成的、要测试是否为口令的词汇输出到标准输出上

\--restore\[=NAME\]

恢复被中断的破解过程，从指定文件或默认为 $JOHN/john.rec 的文件中读取破解过程的状态信息

\--session=NAME

将新的破解会话命名为 NAME ，该选项用于会话中断恢复和同时运行多个破解实例的情况

\--status\[=NAME\]

显示会话状态

\--make-charset=FILE

生成一个字符集文件，覆盖 FILE 文件，用于增量模式

\--show

显示已破解口令

\--test

进行基准测试

\--users=\[-\]LOGIN|UID\[,..\]

选择指定的一个或多个账户进行破解或其他操作，列表前的减号表示反向操作，说明对列出账户之外的账户进行破解或其他操作

\--groups=\[-\]GID\[,..\]

对指定用户组的账户进行破解，减号表示反向操作，说明对列出组之外的账户进行破解。

\--shells=\[-\]SHELL\[,..\]

对使用指定 shell 的账户进行操作，减号表示反向操作

\--salts=\[-\]COUNT

至少对 COUNT 口令加载加盐，减号表示反向操作

\--format=NAME

指定密文格式名称，为 DES/BSDI/[MD5](https://so.csdn.net/so/search?q=MD5&spm=1001.2101.3001.7020)/BF/AFS/LM 之一

\--save-memory=LEVEL

设置内存节省模式，当内存不多时选用这个选项。 LEVEL 取值在 1~3 之间

下面我们以破解 `Linux` 用户密码为例子简单讲解一些 `John the Ripper` 的用法。

### 三、John the Ripper 破解 Linux 用户登录密码

#### 环境准备

*   `Debian/Ubuntu`
*   `John the Ripper` 工具
*   `/etc/passwd`
*   `/etc/shadow`

> 注：`/etc/passwd` 和 `/etc/shadow` 两个文件包含了用户的信息和密码 `hash` 值

#### 破解过程

**1、创建一个测试用户**

我们可以添加一个测试用户 `test` ，并把它密码设置为 `password` 。

创建新用户test：

    sudo useradd -m test -G sudo -s /bin/zsh
           

`zsh` 是我当前默认的 `shell` ，你可以通过如下命令查看你当前的 `shell` ：

    echo $SHELL
           

![](https://ask.qcloudimg.com/http-save/yehe-1564476/3cfstwq0hg.png?imageView2/2/w/1620)

设置 `test` 用户的密码：

    sudo passwd test
           

出现如下信息，即说明成功了。

![](https://ask.qcloudimg.com/http-save/yehe-1564476/buusg93alu.png?imageView2/2/w/1620)

**2、利用 `unshadow` 组合 `/etc/passwd` 和 `/etc/shadow` 两个文件**

`unshadow` 命令会将 `/etc/passwd` 的数据和 `/etc/shadow` 的数据结合起来，创建 `1` 个含有用户名和密码详细信息的文件。

我们可以通过以下命令将这两个文件结合起来：

    cd ~
    sudo unshadow /etc/passwd /etc/shadow > test_passwd
           

**3、使用 `John the Ripper` 破解 `Linux` 用户登录密码**

我们这时候还需要一个字典，由于 `John` 自带了一个字典，字典位于 `/usr/share/john/password.lst` 。当然你也可以自行去网上选择适合你的字典。

我们可以开始对Linux登录用户名和密码进行破解：

    john --wordlist=/usr/share/john/password.lst test_passwd
           

破解结果如下：

![](https://ask.qcloudimg.com/http-save/yehe-1564476/hstq5yghzm.png?imageView2/2/w/1620)

我们可以看到， `test_passwd` 文件中存在的三个用户名 `root` ，`test` ， `python` 的密码，均被破解了。

我们可以查看破解信息：

    john --show test_passwd
           

![](https://ask.qcloudimg.com/http-save/yehe-1564476/0dyet5hbus.png?imageView2/2/w/1620)

以上是关于 `John the Ripper` 其中一个简单地使用，更多高级用法，像暴力破解，增量模式等等，请参看官方文档：[https://www.openwall.com/john/doc/MODES.shtml](https://www.openwall.com/john/doc/MODES.shtml)

[查看原网页: blog.csdn.net](https://blog.csdn.net/i_can1/article/details/107227565?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165655324816781435457364%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165655324816781435457364&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-107227565-null-null.142^v26^control,157^v15^new_3&utm_term=john&spm=1018.2226.3001.4187)





# Quote
