最近打算开一个性能好的VPS比如DO，然后用SS自己搭建一个VPN用。在搜集教程的时候，发现有人说DO开了VPS速度并不好，连720P的YOUTUBE都卡，后来有人提醒说由于国际带宽有限，造成720P卡顿的原因是TCP拥堵。所以才有了下面这个小教程，就是启用TCP_HYBLA模块，开解决TCP拥堵的问题。DigitalOcean、RamNode和BlueVM的KVM都可以正常加载此模块。

内核参数优化
--

首先，将 Linux 内核升级到 3.5 或以上。

### 第一步，增加系统文件描述符的最大限数 ###
编辑文件 `limits.conf`

	vi /etc/security/limits.conf

增加以下两行

	* soft nofile 51200
	* hard nofile 51200

启动shadowsocks服务器之前，设置以下参数

	ulimit -n 51200

### 第二步，优化TCP,优化内核中的拥塞算法 ###

> inux内核中提供了若干套TCP拥塞控制算法，这些算法各自适用于不同的环境。  
1）reno是最基本的拥塞控制算法，也是TCP协议的实验原型。  
2）bic适用于rtt较高但丢包极为罕见的情况，比如北美和欧洲之间的线路，这是2.6.8到2.6.18之间的Linux内核的默认算法。  
3）cubic是修改版的bic，适用环境比bic广泛一点，它是2.6.19之后的linux内核的默认算法。  
4）hybla适用于高延时、高丢包率的网络，比如卫星链路——同样适用于中美之间的链路。  

首先，`OpenVZ`的VPS可以不用继续了。对内核的操作权限太低，没法添加相关模块。  

这个教程只针对`Xen`、`KVM`!!这个教程只针对`Xen`、`KVM`!!这个教程只针对`Xen`、`KVM`!!

#### 加载tcp_hybla模块 ####
1、加载tcp_hybla模块(OpenVZ在这一步就会报错)：

	/sbin/modprobe tcp_hybla

2、然后查看是否已经正常加载：

	lsmod |grep hybla

如果你的内核版本较新，比如CentOS 6.x的2.6.32，则可以用下列命令查看当前可用的拥堵算法，里面应该有hybla了：

	sysctl net.ipv4.tcp_available_congestion_control

建议想折腾的可以先买个DigitalOcean的，毕竟可以按小时计费，不浪费钱。
<!--more-->

#### 修改`/etc/sysctl.conf` ####

	vi /etc/sysctl.conf

将下述内容（综合了几个博客的内容）添加入sysctl.conf文件：

	fs.file-max = 51200
	#提高整个系统的文件限制
	net.ipv4.tcp_syncookies = 1
	#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
	net.ipv4.tcp_tw_reuse = 1
	#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
	net.ipv4.tcp_tw_recycle = 0
	#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭；
	#为了对NAT设备更友好，建议设置为0
	net.ipv4.tcp_fin_timeout = 30
	#修改系统默认的 TIMEOUT 时间。
	net.ipv4.tcp_keepalive_time = 1200
	#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
	net.ipv4.ip_local_port_range = 10000 65000
	#表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为10000到65000。（注意：这里不要将最低值设的太低，否则可能会占用掉正常的端口！）
	net.ipv4.tcp_max_syn_backlog = 8192
	#表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
	net.ipv4.tcp_max_tw_buckets = 5000
	#表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。
	#额外的，对于内核版本新于3.7.1的，我们可以开启tcp_fastopen：
	#net.ipv4.tcp_fastopen = 3
	#使用uname -r或uname -a命令查看内核版本

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
	# 设置高延迟hybal，低延迟cubic
	net.ipv4.tcp_congestion_control=hybla
	
	#定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数。backlog默认会限制到128，而nginx默认为511。较大内存的Linux，65535数值一般就可以了。
	net.core.somaxconn = 4096

可参考：https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks

精简版本（去掉注释）：

	fs.file-max = 51200
	net.ipv4.tcp_syncookies = 1
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_tw_recycle = 0
	net.ipv4.tcp_fin_timeout = 30
	net.ipv4.tcp_keepalive_time = 1200
	net.ipv4.ip_local_port_range = 10000 65000
	net.ipv4.tcp_max_syn_backlog = 8192
	net.ipv4.tcp_max_tw_buckets = 5000
	#net.ipv4.tcp_fastopen = 3

	net.core.rmem_max = 67108864
	net.core.wmem_max = 67108864
	net.ipv4.tcp_rmem = 4096 87380 67108864
	net.ipv4.tcp_wmem = 4096 65536 67108864
	net.core.netdev_max_backlog = 250000
	net.ipv4.tcp_mtu_probing=1
	net.ipv4.tcp_congestion_control=hybla
	#net.ipv4.tcp_congestion_control=cubic
	
	net.core.somaxconn = 4096
	
保存后，可以用下面命令让设置立即生效：

	sysctl -p
	
#### 设置开机后自动加载tcp_hybla模块 ####
刚才第一步里加载的模块只是暂时的，开机后还得重新加载。怎样自动加载呢？

以`CentOS`为例，在`/etc/sysconfig/modules`目录下添加一个`hybla.modules`文件，

	cd /etc/sysconfig/modules
	vi hybla.modules

并且写入以下内容：

	#!/bin/sh
	/sbin/modprobe tcp_hybla
	
然后设置下可执行属性，以便于系统在开机时自动执行：

	chmod +x hybla.modules

锐速
--
- 锐速[官网](http://www.serverspeeder.com/)
- linux版[下载地址](http://my.serverspeeder.com/w.do?m=lsl)

自行参照说明安装。

net-speeder
--
net-speeder 原理非常简单粗暴，就是发包翻倍，这会占用大量的国际出口带宽，本质是损人利己，不建议使用。

如果是debian系统：
```
wget --no-check-certificate https://raw.githubusercontent.com/tennfy/debian_netspeeder_tennfy/master/debian_netspeeder_tennfy.sh
chmod a+x debian_netspeeder_tennfy.sh
bash debian_netspeeder_tennfy.sh
```

教程[传送门](http://wuchong.me/blog/2015/02/02/shadowsocks-install-and-optimize/)

如何安装ss
--
脚本一键安装python版ss：[传送门](http://teddysun.com/342.html)  
其他版本上面文章后面有教程链接。

安装完并优化后，访问下youtube视频吧！

比较靠谱、口碑比较好的VPS商家顺序：[Linode](https://www.linode.com)>[DigitalOcean](https://cloud.digitalocean.com)>[VULTR](https://www.vultr.com)

链接
--
原文地址：[传送门](http://www.667887.net/use-tcp-hybla-algorithm-to-improve-foreign-vps-access.html)

参考链接：

- [wuchong传送门](http://wuchong.me/blog/2015/02/02/shadowsocks-install-and-optimize/)
- [quericy传送门](http://quericy.me/blog/495)-含linode编译更新内核教程
- [tennfy传送门](http://www.tennfy.com/1978.html)
