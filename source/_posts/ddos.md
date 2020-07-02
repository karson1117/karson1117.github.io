---
title: Linux下防御/减轻DDOS攻击 
date: 2017-03-03 13:30:03
tags:
	- vps
	- ddos
categories:
	- Linux
	- 安全
---
好久没管我的vps了，直到前天发现vpn居然用不了了，一脸懵逼。
登录后台就发现可能是被攻击了，导致被官方限制使用。
![](/img/articleImg/vpsproblem.jpg)
<!--more-->
被莫名的攻击后也是张二和尚摸不着头脑。
还能咋办叻?Goole~&Baidu~
然后就开始了如下操作，也不知有没有用，O(∩_∩)O哈哈~

## DDoS deflate介绍

>DDoS deflate是一款免费的用来防御和减轻DDoS攻击的脚本。它通过>netstat监测跟踪创建大量网络连接的IP地址，在检测到某个结点超过
>预设的限 制时，该程序会通过APF或IPTABLES禁止或阻挡这些IP.

### 如何确认是否受到DDOS攻击？

由于我是直接重装了系统，所以这一步仅供参考
{% codeblock %}
netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
{% endcodeblock %}

据说：执行后，将会显示服务器上所有的每个IP多少个连接数。

### 1.安装DDoS deflate
{% codeblock %}
wget http://www.inetbase.com/scripts/ddos/install.sh   //下载DDoS  deflate
chmod 0700 install.sh    //添加权限
./install.sh             //执行
{% endcodeblock %}

### 2.配置DDoS deflate
DDoS deflate的默认配置位于/usr/local/ddos/ddos.conf
内容如下
![](/img/articleImg/ddosconf.png)

查看/usr/local/ddos/ddos.sh文件的第117行
(vim模式下 跳转命令：行号 即可)
{% codeblock %}
netstat -ntu | awk ‘{print $5}’ | cut -d: -f1 | sort | uniq -c | sort -nr > $BAD_IP_LIST
{% endcodeblock %}
修改为以下代码即可！
{% codeblock %}
netstat -ntu | awk ‘{print $5}’ | cut -d: -f1 | sed -n ‘/[0-9]/p’ | sort | uniq -c | sort -nr > $BAD_IP_LIST
{% endcodeblock %}