---
title: VPS搭建多端口VPN
date: 2016-12-20 16:37:45
tags: 
	- vps
	- vpn
categories:
	- 其他
---
>为了防止世界被破坏，为了守护世界的和平!
>贯彻爱与真实的邪恶，可爱又迷人的反派角色!
>世界辣么大~还不去看看！

![](/img/articleImg/naruto.jpg)
<!--more-->
## 所需
1.他乡vps一台
2.xshell用于远程登录
3.shadowsocks应用程序。[点这里](https://github.com/shadowsocks/shadowsocks-windows/releases)

## Ready.
1.本人装备的是搬瓦工的vps,好处是可以无限制的重装系统。
2.[购买VPS](http://hostingset.com/)
3.打开xshell连接
![](/img/articleImg/xshell.png)

## GO！！
1.安装所需组件
{% codeblock %}
 yum install m2crypto python-setuptools
 easy_install pip
 pip install shadowsocks
{% endcodeblock %}

2.新建配置文件：shadowsocks.json
{% codeblock %}
{
    "server":"23.105.215.43",   #你服务器的ip
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{           #端口以及对应的密码
         "9000":"password",
         "9001":"password",
         "9002":"password"
    },
    "timeout":300,
    "method":"rc4-md5",         #选择的加密方式
    "fast_open": false
}
{% endcodeblock %}
3添加
{% codeblock %}
 ssserver -c /etc/shadowsocks.json

 ssserver -c /etc/shadowsocks.json -d start  #后台运行
{% endcodeblock %}
4.开启shadowsocks应用程序，输入ip,端口号,密码。就OK啦~
5.Android用户想要连接本文配置的vpn,请搜下图app.
![](/img/articleImg/shadowsocks.png)