---
title: VPS安装及配置nginx
date: 2017-01-04 13:43:14
tags: 
	- nginx
categories:
	- Linux	
---

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。
![](/img/articleImg/nginx.jpg)
<!--more-->
## 环境确认
系统环境：Centos 6
安装方式：源码编译安装 [几种Linux软件的安装方法](http://www.285868.com/a/xtjc/5635.html)
安装位置：/usr/local/nginx
## 安装前提
在安装nginx前，需要确保系统安装了g++、gcc、openssl-devel、pcre-devel和zlib-devel软件。
## 开始安装
进入安装目录
{% codeblock %}
 cd /usr/local/src
{% endcodeblock %}
### 安装pcre(用于Nginx的HTTP Rewrite 模块)
>注：若使用wget方式下载安装包可能会出现地址变更导致下载失败。
>另一种方式：xftp可进行文件传输，本地下载好安装包后，利用xftp传输即可。

{% codeblock %}
cd /usr/local/src 
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz  	   
tar -zxvf pcre-8.38.tar.gz #解压文件
mv pcre-8.38   pcre #修改解压后文件夹名称
cd pcre
./configure 		#准备编译
make & make install	#编译及安装
{% endcodeblock %}

### 安装zilb(一个压缩和解压模块)
{% codeblock %}
cd /usr/local/src
wget http://zlib.net/zlib-1.2.8.tar.gz
tar -zxvf zlib-1.2.8.tar.gz
mv zlib-1.2.8 zlib
cd zlib
./configure
make & make install
{% endcodeblock %}
### 安装SSl模块
>openssl是为网络通信提供安全及数据完整性的一种安全协议，囊括了主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议，并提供了丰富的应用程序供测试或其它目的使用。

{% codeblock %}
cd /usr/local/src
wget http://www.openssl.org/source/openssl-1.0.1c.tar.gz
tar -zxvf openssl-1.0.1c.tar.gz
mv openssl-1.0.1c  openssl
./config
make & make install
{% endcodeblock %}

### 安装Nginx
{% codeblock %}
cd /usr/local/src
wget http://nginx.org/download/nginx-1.9.9.tar.gz
tar -zxvf nginx-1.9.9.tar.gz
cd nginx-1.9.9

./configure --sbin-path=/usr/local/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-http_v2_module \
--with-pcre=/usr/local/src/pcre \
--with-zlib=/usr/local/src/zlib \
--with-openssl=/usr/local/src/openssl

make & make install

cd /usr/local/nginx #进入nginx目录
nginx				#启动nginx
{% endcodeblock %}
启动后浏览器导航到http://IP 就可以看到默认的欢迎界面了

### Nginx常用命令
{% codeblock %}
 nginx -s stop停止nginx
 nginx 运行nginx
 nginx -s reload 重启nginx
 nginx -t 测试nginx
{% endcodeblock %}
### nginx加入到环境变量
{% codeblock %}
 vim /etc/profile
{% endcodeblock %}
尾行添加
{% codeblock %}
PATH=$PATH:/usr/local/nginx #nginx安装目录
export PATH 
{% endcodeblock %}
保存关闭后运行
{% codeblock %}
 source /etc/profile
{% endcodeblock %}
### 修改网站默认根目录路径
>网站默认根目录放在/usr/local/nginx/html

{% codeblock %}
vim /usr/local/nginx/conf/nginx.conf
{% endcodeblock %}

{% codeblock %}
server {
	listen       80;
	server_name  你的IP;
	#charset koi8-r;

	#access_log  logs/host.access.log  main;
	location / {
	root  /www/blog/public;
	index index.html index.htm;
	}
	.
	.
	.
}
{% endcodeblock %}
root添加自己想要的根目录，重启nginx生效。