# TCP是怎样工作的

![cover](fig/cover.jpg)

[TCP是怎样工作的的支持页面](https://www.ituring.com.cn/book/2851)．

## 环境搭建

本书使用VirtualBox、Vagrant以及X Server的组合搭建模拟环境。接下来介绍环境构建的方法。

### 验证环境

已在以下的环境中进行了验证。模拟基本上是通过虚拟机进行的，因此只需搭建出如下所示的VirtualBox，Vagrant和X Server的环境，masOS以外的系统也可以无碍地完成模拟。

- OS：masOS Mojave 10.14.3
- 处理器：2.9GHz Intel Core i7
- 内存：16GB 2133MHz LPDDR3
- VirtualBox：6.0.4r128413
- Vagrant：2.2.4

此外，译者在如下环境下进行了验证：
- OS：Windows 10 专业版 21H2 19044.2604
- 处理器：Intel(R) Core(TM) i9-9900K CPU @ 3.60GHz
- 内存：32.0 GB 3200MHz
- VirtualBox：6.1.28 r147628 (Qt5.6.2)
- Vagrant：2.3.4
- MobaXterm Personal Edition 23.0 Build 5042

已在如下的VirtualBox虚拟机环境下进行了验证。
- Ubuntu: 16.04
- WireShark: 2.6.5
- ns-3: 3.27
- python: 3.5.2
- gcc: 5.4.0
- make: 4.1

此外，到2019年4月1日为止，ns-3的安装向导并没有适配Ubuntu 18.04，因此本书使用Ubuntu 16.04。另外，到2019年4月1日为止，第5章和第6章使用的CUBIC和BBR模块没有适配ns-3.28以上的版本，因此本书使用ns-3.27。

请注意，由于要运行4GB内存的虚拟机，因此需要准备拥有4GB以上内存的物理机。

### Oracle VM VirtualBox

到2019年4月1日为止，从VirtualBox的[Web官网](https://www.virtualbox.org/)的`Download`页面上可以下载到与宿主OS适配的安装包。运行安装程序可以完成VirtualBox的安装。

macOS系统下，打开终端程序（例如Terminal.app），执行以下命令，就可以确认VirtualBox的安装情况。请注意，根据所使用的环境不同，终端上显示的版本号可能不一样。

```shell
$ VBoxManage -v
> 6.0.4r128413
```

### Vagrant

在2019年4月1日的时间点，点击Vagrant的[Web网站](https://www.vagrantup.com/)的[Download]按钮，页面会跳转到安装包的下载界面。请根据所使用的环境，选择对应的安装包，下载完成之后执行安装程序完成安装。

macOS系统下，请在终端程序(Terminal.app等)中执行以下命令。如果有输出对应的版本号，则说明Vagrant已经安装完毕。请注意，根据所使用的环境不同，终端上显示的版本号可能不一样。

```shell
$ vagrant -v
> Vagrant 2.2.4
```

### X server

本书在虚拟机OS上使用X Window System运行Wireshark，因此需要在物理机OS上搭建X Server环境。

在2019年4月1日的时间点，macOS X Serra系统下，可以通过[XQuartz项目](https://www.xquartz.org/)的Web网站获取X11 Server（X11.app）。请注意，如果使用其他OS，获取的方式有所不同。

Windows 操作系统可以使用[MobaXterm](https://mobaxterm.mobatek.net/)

为了验证X Server的运行情况，请启动虚拟机上的GUI应用程序。首先，请打开已下载的本书源代码，定位到wireshark/vagrant目录，执行以下命令。

```shell
$ vagrant up
```

这里Windows操作系统可能会报错，如下：
```bash
process_builder.rb:44:in `encode!': "\\xE5" to UTF-8 in conversion from ASCII-8BIT to UTF-8 to UTF-16LE (Encoding::UndefinedConversionError)
```
请找到报错信息的process_builder.rb 44行，编码位置换成一下：
```bash
#newstr.encode!('UTF-16LE')
newstr.encode!('UTF-16LE', invalid: :replace, undef: :replace, replace: '?')
```

第4章所使用的Wireshark虚拟环境就搭建完成了（可能会花费一点时间）。接下来执行下面的命令，进行SSH登录，启动`xeyes`。

注意Windows操作系统，在`MobaXterm`中运行`vagrant ssh guest1`之前，请先运行`cmd.exe`，启动命令行，然后再运行`vagrant ssh guest1`，否则会报错。

```shell
$ vagrant ssh guest1

> Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic x86_64)
>
> * Documentation:  https://help.ubuntu.com
> * Management:     https://landscape.canonical.com
> * Support:        https://ubuntu.com/advantage
>
> Get cloud support with Ubuntu Advantage Cloud Guest:
> http://www.ubuntu.com/business/services/cloud
>
> 0 packages can be updated.
> 0 updates are security updates.
>
> New release '18.04.1 LTS' available.
> Run 'do-release-upgrade' to upgrade to it.

vagrant@guest1:~$ xeyes
```
请确认是否有下图所示的两个眼球出现。

![xeyes](fig/xeyes.png)

请执行以下命令，暂时登出，停止运行虚拟机。

```shell
vagrant@guest1:~$ exit
$ vagrant halt
```

## 模拟环境的确认

本书使用Wireshark和ns3进行模拟。接下来确认一下是否环境构建是否完成。

### WireShark

当确认完VirtualBox和Vagrant的环境已经搭建完成之后，请将[此Github代码库](https://github.com/ituring/tcp-book)克隆到任意目录。接下来打开`wireshark/vagrant` 目录，执行`vagrant up`命令。这样，就可以在两台虚拟机上完成Ubuntu 16.04的环境搭建。

```shell
$ git clone https://github.com/ituring/tcp-book.git
$ cd tcp-book/wireshark/vagrant
$ vagrant up
```

使用以下的命令，ssh连接到Guest操作系统上。在登录消息显示之后，命令行提示会变成`vagrant@guest1:~$`。

注意Windows操作系统，在`MobaXterm`中运行`vagrant ssh guest1`之前，请先运行`cmd.exe`，启动命令行，然后再运行`vagrant ssh guest1`，否则会报错。

```shell
$ vagrant ssh guest1

> Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic x86_64)
>
> * Documentation:  https://help.ubuntu.com
> * Management:     https://landscape.canonical.com
> * Support:        https://ubuntu.com/advantage
>
> Get cloud support with Ubuntu Advantage Cloud Guest:
> http://www.ubuntu.com/business/services/cloud
>
> 0 packages can be updated.
> 0 updates are security updates.
>
> New release '18.04.1 LTS' available.
> Run 'do-release-upgrade' to upgrade to it.

vagrant@guest1:~$
```

启动Wireshark。

```shell
vagrant@guest1:~$ wireshark
```

![wireshark](fig/wireshark.png)

当看到如图4.18一样的画面时，则表示Wireshark已经启动完成，准备工作也就完成了。此时请先登出，并关闭虚拟机。

```shell
vagrant@guest1:~$ exit
$ vagrant halt
```

### ns3

当确认已经准备好VirtualBox和Vagrant的环境之后，请将[此Github代码库](https://github.com/ituring/tcp-book)克隆到任意目录。打开其中的`ns3/vagrant`目录，执行`vagrant up`命令。如此一来，就在虚拟机上完成了安装Ubuntu 16.04，并搭建ns-3的过程。另外，在2019年4月1日的时间点，第5章和第6章所使用的CUBIC和BBR模块不支持ns-3.28以上版本，因此本书使用ns-3.27版本。由于搭建ns-3环境相当花时间，请务必耐心等待 。


```shell
$ git clone https://github.com/ituring/tcp-book.git
$ cd tcp-book/ns3/vagrant
$ vagrant up
```

接下来，ssh连接到Guest操作系统上。

注意Windows操作系统，在`MobaXterm`中运行`vagrant ssh`之前，请先运行`cmd.exe`，启动命令行，然后再运行`vagrant ssh`，否则会报错。

```shell
$ vagrant ssh
> Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-139-generic x86_64)
>
>  * Documentation:  https://help.ubuntu.com
>  * Management:     https://landscape.canonical.com
>  * Support:        https://ubuntu.com/advantage
>
>   Get cloud support with Ubuntu Advantage Cloud Guest:
>     http://www.ubuntu.com/business/services/cloud
>
> 13 packages can be updated.
> 6 updates are security updates.
>
> New release '18.04.1 LTS' available.
> Run 'do-release-upgrade' to upgrade to it.
>
>
vagrant@ubuntu-xenial:~$
```

如上所述，当提示出现`vagrant@ubuntu-xenial:~$`之后，SSH就连接成功了。

## 勘误表

本书的勘误信息在以下的页面中公开。

https://www.ituring.com.cn/book/2851

如果您发现了本页面尚未公布的问题，请在[本书的支持页面](https://www.ituring.com.cn/book/2851)予以告知。

## 作者介绍

### 安永辽真

2011年毕业于东京大学工学部，2013年硕士毕业于东京大学研究生院工学系，同年入职日本电信电话株式会社，2016年被派往诺基亚贝尔实验室进修。主要从事计算机网络模型的研究。 

### 中山悠
2008年毕业于东京大学，入职日本电信电话株式会社，2018年博士毕业于东京大学电子信息专业，目前在东京农工大学担任副教授，研究移动计算、低延迟网络和物联网等。

### 丸田一辉

2008年毕业于九州大学，入职日本电信电话株式会社，2016年博士毕业于九州大学信息智能工学专业，2017年3月成为千叶大学助教，研究无线网络中的抗干扰技术。曾获得日本电子信息通信学会（IEICE）论文奖、日本无线电通信系统（RCS）研究会最优秀贡献奖等。

### 尹修远

毕业于华中科技大学，现从事客户端开发工作。曾任职腾讯游戏平台，从事网络加速相关技术研发，对TCP/IP等网络技术有自己独到的见解。
