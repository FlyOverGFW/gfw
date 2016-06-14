**2015.11.21更新** 参考：https://github.com/phuslu/goproxy/issues/186

搬瓦工不支持锐速、不支持TCP Fast Open、不支持hybla，加入hybla参数是为了其他服务器也能使用这篇文章的内容，其实那个参数对搬瓦工来说，然并卵

**1 加密方式**
加密方式改为CR4-MD5，相对aes-256-cfb来说可能加密方式更弱，但加密速度是后者的好几倍，对于一般用户来说，没有影响但能提高速度。

服务器端口一定要使用默认的443端口（很重要）

**2 系统文件描述符**

```
vi /etc/security/limits.conf
```
增加以下两行
```
* soft nofile 51200
* hard nofile 51200
```
启动shadowsocks服务器之前，设置以下参数（这一步是Shadowsocks官文上讲的，但实际上上面两步就是执行下面这一步，所以，下面的参数可以跳过了）
```
ulimit -n 51200
```

**3 更新sysctl和modprobe**

(非openvz平台跳过这一步，openvz据说改内核参数然并卵)

```
rm -f /sbin/sysctl
ln -s /bin/true /sbin/sysctl
rm -f /sbin/modprobe
ln -s /bin/true /sbin/modprobe
```

**4 配置文件 sysctl.conf**

```
vi /etc/sysctl.conf

fs.file-max = 51200
net.ipv4.conf.lo.accept_redirects=0
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.eth0.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_congestion_control = hybla
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_rmem  = 32768 436600 873200
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_timestsmps = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_tw_buckets = 9000
net.ipv4.tcp_max_syn_backlog = 65536
net.ipv4.tcp_mem = 94500000 91500000 92700000
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_wmem = 8192 436600 873200
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 32768
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864

sysctl -p
```

**5 net-speeder**

```
wget --no-check-certificate https://gist.github.com/LazyZhu/dc3f2f84c336a08fd6a5/raw/d8aa4bcf955409e28a262ccf52921a65fe49da99/net_speeder_lazyinstall.sh
sh net_speeder_lazyinstall.sh
echo 'nohup /usr/local/net_speeder/net_speeder venet0 "ip" >/dev/null 2>&1 &' >> /etc/rc.local
```
如果是debian，可以用下面这个：
```
wget --no-check-certificate https://raw.githubusercontent.com/tennfy/debian_netspeeder_tennfy/master/debian_netspeeder_tennfy.sh
chmod a+x debian_netspeeder_tennfy.sh
bash debian_netspeeder_tennfy.sh
```

官方参考：https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks

**以下是旧的内容 @15.11.21**

---

搬瓦工shadowsocks优化方案
----
![](http://7xiohk.com1.z0.glb.clouddn.com/img/bandwagon.png)
> 按照shadowsocks官网上的来http://shadowsocks.org/en/config/advanced.html 因为我的VPS用的是CentOS，所以在最后执行的时候会报错，可以照着这个来解决http://www.111cn.net/sys/CentOS/54693.htm
<!--more-->

以下两个教程，暂时只尝试过第一个，第二个未亲测，值得尝试！

No.1 教你如何在搬瓦工vps优化shadowsocks（[原文](http://www.40do.com/archives/7411.html)﻿）
-- 
### 步骤 ###

	rm -f /sbin/modprobe
	ln -s /bin/true /sbin/modprobe
	rm -f /sbin/sysctl
	ln -s /bin/true /sbin/sysctl
	vi /etc/sysctl.conf

增加以下内容

	fs.file-max = 51200
	net.ipv4.tcp_syncookies = 1
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_tw_recycle = 0
	net.ipv4.tcp_fin_timeout = 30
	net.ipv4.tcp_keepalive_time = 1200
	net.ipv4.ip_local_port_range = 10000 65000
	net.ipv4.tcp_max_syn_backlog = 8192
	net.ipv4.tcp_max_tw_buckets = 5000
	net.ipv4.tcp_fastopen = 3
	# increase TCP max buffer size settable using setsockopt()
	net.core.rmem_max = 67108864
	net.core.wmem_max = 67108864
	# increase Linux autotuning TCP buffer limit
	net.ipv4.tcp_rmem = 4096 87380 67108864
	net.ipv4.tcp_wmem = 4096 65536 67108864
	# increase the length of the processor input queue
	net.core.netdev_max_backlog = 250000
	# recommended for hosts with jumbo frames enabled
	net.ipv4.tcp_mtu_probing=1
	#增加以上内容

保存后

	sysctl -p
	reboot

### 安装Net_speeder ###

由于TCP的特性导致，每个包发一次，容易掉包。干脆就一个包发两次，有效降低掉包率。
[Net_speeder的Github项目地址](https://github.com/snooda/net-speeder)

重启回来继续执行以下命令

	apt-get install libnet1
	apt-get install libpcap0.8
	apt-get install libnet1-dev
	apt-get install libpcap0.8-dev
	wget https://net-speeder.googlecode.com/files/net_speeder-v0.1.tar.gz
	tar zxvf net_speeder-v0.1.tar.gz
	sh ./net_speeder/build.sh -DCOOKED
	./net_speeder/net_speeder venet0 "tcp" > netlog &

### 注意！关于`apt-get install libnet1`报错 ###

先执行一下

	apt-get update
	apt-get upgrade

更新完成再安装libnet1试试看..  
我在搬瓦工上实测不会报错..  

**还有..如果是centos的话..是不能这样安装的..**

#### debian/ubuntu安装 ####
安装libnet：apt-get install libnet1  
安装libpcap： apt-get install libpcap0.8 
 
编译安装需要`libnet`和`libpcap`对应的dev包 
 
安装libnet-dev：apt-get install libnet1-dev  
安装libpcap-dev： apt-get install libpcap0.8-dev  

#### centos安装 ####
下载`epel`：[https://fedoraproject.org/wiki/EPEL/zh-cn](https://fedoraproject.org/wiki/EPEL/zh-cn) 
 
例：CentOS6 64位：

	wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

如果是centos5，则在`epel/5/`下  

然后安装`epel`：`rpm -ivh epel-release-X-Y.noarch.rpm`（实际上CentOS 用户可以直接通过 `yum install epel-release` 安装并启用 EPEL 源。）  

然后即可使用yum安装`libnet`、`libcap`：  

	yum install libnet libpcap libnet-devel libpcap-devel

下载net-speeder：[项目地址](https://github.com/snooda/net-speeder)
 
~~wget https://net-speeder.googlecode.com/files/net_speeder-v0.1.tar.gz 
解压缩：tar -zxvf net_speeder-v0.1.tar.gz~~
  
**使用方法(需要root权限启动）**
参数：./net_speeder 网卡名 加速规则（bpf规则）  
最简单用法： # ./net_speeder venet0 “ip” 加速所有ip协议数据  

No.2 编译内核（[原文](http://quericy.me/blog/495)）
--

对于`OpenVZ`,如果直接使用以上方法修改,当`sysctl -p`会发现一堆的permission denied出错信息.谷歌了下,据说是因为OpenVZ模版的限制比较多直接修改sysctl会被拒绝，因此以上优化方案在OpenVZ下不可用.

**优化拥塞算法Hybla:**这部分参照了V2EX上的tcp_hybla 编译内核模块的教程.

这一步本人在Linode上实验成功.(Linode是Xen的),系统为Ubuntu,其他系统没有尝试.

当然据说Digital Ocean (DO是KVM模板)上的ubuntu官方内核自带了,**可略去编译部分,直接调用hybla算法**。(补充:经本人测试,Digital Ocan上,直接在/etc/sysctl.conf文件中加入hybla参数就可以直接调用了.如果你是Digital Ocan的话,恭喜你,直接跳到下面方法的第九步)

教程原文转载以作备用:

1.查看你的机器内核版本：

	uname -r

显示结果如: 3.11.6-x86_64-linode35

2.去 https://www.kernel.org/pub/linux/kernel/v3.0/ 下载相同版本的源码到任意目录，解压

	mkdir /root/mykernel
	cd /root/mykernel
	wget https://www.kernel.org/pub/linux/kernel/v3.0/linux-3.11.6.tar.gz
	tar xzvf linux-3.11.6.tar.gz

3.安装内核编译工具

	apt-get update && apt-get install -y build-essential libncurses5-dev

4.复制Linode原来的内核编译配置文件到源码根目录，在`CONFIG_TCP_CONG_CUBIC=y`下面增加一行 `CONFIG_TCP_CONG_HYBLA=y`，再生成编译模块需要的内核

	cd linux-3.11.6
	zcat /proc/config.gz > .config
	vi .config
查找`CONFIG_TCP_CONG_CUBIC=y`，在下面增加一行`CONFIG_TCP_CONG_HYBLA=y`，保存

	make

5.耐心等待编译内核完成，单核编译大约需15分钟，完成后，进入模块所在的目录，编写Makefile

	cd net/ipv4/
	mv Makefile Makefile.old
	vi Makefile

以下是`Makefle`的内容，注意要把`KDIR`修改为你自己的源码路径，其他则照抄就可以了

	Makefile for tcp_hybla.ko
	obj-m := tcp_hybla.o
	KDIR := /root/mykernel/linux-3.11.6
	PWD := $(shell pwd)
	default:
	$(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules

6.进入源码根目录，编译模块

	cd /root/mykernel/linux-3.11.6/
	make modules

7.进入到模块所在目录，复制生成的 tcp_hybla.ko 到加载目录，测试加载模块

	cd /root/mykernel/linux-3.11.6/net/ipv4
	cp tcp_hybla.ko /root/mykernel/
	cd /root/mykernel/

加载前

	sysctl net.ipv4.tcp_available_congestion_control
	net.ipv4.tcp_available_congestion_control = cubic reno
	insmod tcp_hybla.ko
加载后

	sysctl net.ipv4.tcp_available_congestion_control
	net.ipv4.tcp_available_congestion_control = cubic reno hybla

设置hybal为优先

	sysctl net.ipv4.tcp_congestion_control=hybla
8.设置开机自动加载模块，把`tcp_hybla.ko` 复制到`/lib/modules/3.11.6-x86_64-linode35/kernel/net/ipv4`

	cd /lib/modules/3.11.6-x86_64-linode35
	mkdir -p kernel/net/ipv4
	cd kernel/net/ipv4
	cp /root/mykernel/tcp_hybla.ko ./
	cd /lib/modules/3.11.6-x86_64-linode35
	depmod -a
9.修改`/etc/sysctl.conf`开机自动设置hybal为优先

	vim /etc/sysctl.conf
	net.ipv4.tcp_congestion_control = hybla

保存并退出该文件,然后使用以下指令使配置生效:

	sysctl -p

链接
--
[各平台VPS快速搭建SHADOWSOCKS及优化总结](http://quericy.me/blog/495)  
[使用tcp_hybla算法提高国外VPS访问速度](http://i.weaming.info/2015/04/29/Use-tcp-hybla-algorithm-to-improve-access-speed-foreign-VPS/)
