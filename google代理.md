![](http://7xiohk.com1.z0.glb.clouddn.com/img/so-weaming-info.png)

-----

2016.3.6更新 从发行版迁移
==

```
$ nginx -V
nginx version: nginx/1.8.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_spdy_module --with-http_gzip_static_module --with-ipv6 --with-http_sub_module


cd lnmp1.2-full/src;
git clone https://github.com/cuber/ngx_http_google_filter_module
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module

cd nginx*
./configure --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_spdy_module --with-http_gzip_static_module --with-ipv6 --with-http_sub_module \
--add-module=../ngx_http_google_filter_module \
--add-module=../ngx_http_substitutions_filter_module
make

cp -rf objs/nginx /usr/local/nginx/sbin/nginx
```

2015.11.20更新
==

```
mkdir ggproxy; cd ggproxy
apt-get install build-essential git gcc g++ make
wget "http://nginx.org/download/nginx-1.4.7.tar.gz"
wget "ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.36.tar.gz"
wget "https://www.openssl.org/source/openssl-1.0.1j.tar.gz"
wget "http://zlib.net/zlib-1.2.8.tar.gz"
git clone https://github.com/cuber/ngx_http_google_filter_module
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
tar xzvf nginx-1.4.7.tar.gz
tar xzvf pcre-8.36.tar.gz
tar xzvf openssl-1.0.1j.tar.gz
tar xzvf zlib-1.2.8.tar.gz
cd nginx-1.4.7
./configure \
  --prefix=/opt/nginx-1.4.7 \
  --with-pcre=../pcre-8.36 \
  --with-openssl=../openssl-1.0.1j \
  --with-zlib=../zlib-1.2.8 \
  --with-http_ssl_module \
  --add-module=../ngx_http_google_filter_module \
  --add-module=../ngx_http_substitutions_filter_module
make
sudo make install
sudo /opt/nginx-1.4.7/sbin/nginx

重启
sudo /opt/nginx-1.4.7/sbin/nginx -s reload
```

报错： `./configure: 10: .: Can't open auto/options`
---
更改nginx版本为1.4.7得到解决

link：http://ju.outofmemory.cn/entry/207820

报错： `unable to resolve host xxxx`
--
添加解析`/etc/hosts`，比如`127.0.0.1  xxxx`

技巧
--
输入`sudo opt/nginx-1.4.7/sbin/nginx -t`查看当前配置文件路径，注意做好备份。

配置示例
--
详见https://github.com/cuber/ngx_http_google_filter_module/blob/master/README.zh-CN.md

-----
以下为老的记录 
--

~~Demo：<https://so.weaming.info>~~

必备工具
--
1.一台可用的VPS（[虚拟专用服务器](http://baike.baidu.com/view/309631.htm)）；
2.伺服器系统选择：centos

准备工作
--
1.0 登入系统后执行一次源更新

	$ yum update

1.1 安装git

	$ yum install git

1.2 git clone 相关nginx模块：[ngx_http_google_filter_module](https://github.com/cuber/ngx_http_google_filter_module)

	$ git clone https://github.com/cuber/ngx_http_google_filter_module

1.3 git clone 相关nginx模块：[ngx_http_substitutions_filter_module](https://github.com/yaoweibin/ngx_http_substitutions_filter_module)

1.4 安装 [PCRE Library](http://www.pcre.org/)

	$ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.36.tar.gz
	$ tar -xf pcre-8.36.tar.gz
	$ cd pcre-8.36
	$ ./configure
	$ make
	$ make install

1.5 安装 openssl-lib
	
	$ yum install openssl libssl-devlibssl-dev

编译安装的nginx
--
2.1 下载nginx-1.7.8

	$ wget http://nginx.org/download/nginx-1.7.8.tar.gz

2.2 解压nginx-1.7.8

	$ tar -xf nginx-1.7.8.tar.gz

2.3 进入解压目录

	$ cd /root/nginx-1.7.8

2.4 编译 ./configure
	
	$ ./configure --prefix=/usr/local/nginx-1.7.8 --add-module=/root/ngx_http_google_filter_module --add-module=/root/ngx_http_substitutions_filter_module --with-http_ssl_module 

成功画面：
![](http://i.imgur.com/VAKcwOh.png)

2.5 编译并安装

	$ make && make install

2.3 启动 nginx 或关闭 nginx
	
	$ /usr/local/nginx-1.7.8/sbin/nginx
	$ killall nginx

2.4 设置开机启动

	$ vi /etc/rc.local

在文件末尾添加

	$ /usr/local/nginx-1.7.8/sbin/nginx

保存并退出

2.5 查看　nginx 版本(是不是version=1.7.8)

	$ /usr/local/nginx-1.7.8/sbin/nginx -v

2.6 查看nginx 配置(是不是＝步骤2.1设置相关)

	$ /usr/local/nginx-1.7.8/sbin/nginx -V

2.7 配置nginx.conf文件

	$ /usr/local/nginx-1.7.8/sbin/nginx -t #会显示配置是否0k? 

> e.g.
nginx: the configuration file /usr/local/nginx-1.7.8/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx-1.7.8/conf/nginx.conf test is successful

	$ vi /usr/local/nginx-1.7.8/conf/nginx.conf

白话文解释：下面＃1为需要你找到并手动注释掉的代码行（如下操作即可，在相应代码行前加＃号），其他未加＃1的需要你手动添加到,对比下源文件和这里的差别就可以找出来了。大约从34行开始注释吧。

	#1    server {
	#1       listen       80;
	#1     server_name  localhost;
	#1
	#1      #charset koi8-r;
	#1
	#access_log  logs/host.access.log  main;
	
	#1       location / {
	#1         root   html;
	#1       index  index.html index.htm;
	#1 }
	server {
	# ... part of server configuration
	resolver 8.8.8.8 8.8.4.4;
	location / {
	google on;
	google_scholar on;
	# set language to German
	google_language zh-CN;
	}
	# ...
	}
	
	server {
	# ...
	location / {
	google on;
	google_ssl_off "www.google.com";
	}
	# ...
	}
	
	upstream www.google.com {
	server 173.194.38.1:443;
	server 173.194.38.2:443;
	server 173.194.38.3:443;
	server 173.194.38.4:443;
	}
	
	#
	# configuration of vps(us)
	#
	server {
	listen 80;
	server_name www.google.com;
	# ...
	location / {
	proxy_pass https://www.google.com;
	# ...
	}
	#error_page  404              /404.html;
	
	# redirect server error pages to the static page /50x.html
	#
	error_page   500 502 503 504  /50x.html;

博主参照的配置地址：[github: limboshop/ngx_http_google_filter_module](https://github.com/limboshop/ngx_http_google_filter_module/blob/master/nginx.conf)

链接
--
本文参照文章：[limboshop](http://story.limboshop.info/2015/06/17/free-search-project)

项目地址：https://github.com/cuber/ngx_http_google_filter_module

如何安装nginx第三方模块：http://www.ttlsa.com/nginx/how-to-install-nginx-third-modules/

SSL+Nginxの配置相关知识：https://github.com/cuber/ngx_http_google_filter_module/issues/27

参考
--
nginx 配置SSL
![](http://7xiohk.com1.z0.glb.clouddn.com/img/nginx-https-setting.png)
