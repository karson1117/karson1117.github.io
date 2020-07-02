---
title: hexo自动部署至VPS
date: 2017-03-03 14:14:15
tags:
	- vps 
	- git
	- webhook
categories:
	- 其他
---
之前每次写好一篇博文后，都是傻傻的打包利用Xftp手动传至VPS中。简直就是一个字“low~”。自从利用webhook实现自动部署后，腰不酸了，腿不疼了，可以扛两袋米一口气上五楼了~o(￣▽￣)o~ (小装一波~)。
>最终实现：自己电脑上新建文章后，hexo clean && hexo g -d 即可。
>实现原理：
>1.hexo 提交代码渲染后文件至远程仓库(coding)
>2.coding中对应项目配置webhook发送执行请求
>3.vps 接收指定请求执行脚本(拉取最新代码)
<!--more-->

操作分为本机上和vps上的操作。
### hexo本地配置及部署
>hexo是基于nodejs开发的，npm是nodejs的包管理工具
>git用于部署代码
>所以首页应确认本机环境：git，node.js环境
>git,node.js安装可另查资料

{% codeblock %}
#安装hexo命令行工具
npm install hexo-cli -g
#创建blog目录，并初始化hexo项目
hexo init blog
cd blog
hexo new "My First Post"
#生成相关静态文件
hexo g
#启动本地服务，查看效果（http://localhost:4000）
hexo server
{% endcodeblock %}

hexo的git配置，hexo根目录下_config.yml文件中
{% codeblock %}
deploy:
  type: git
  message: update
  repo: git@git.coding.net:letra/hexo.git 
{% endcodeblock %}
{% codeblock %}
#代码部署
hexo deploy
{% endcodeblock %}
如果正确，然后在你的远程Git仓库中就有了hexo项目的相关文件了
重点来了！！！
### 远程仓库配置WebHooks
![](/img/articleImg/webhook.png)
这张图的配置的意思是：当仓库发生push的时候，会发送一个请求到http://karson.cc:4002/webhooks/push/123456。

为了服务端的简易处理，这里没有使用token，而是将url地址当做token，123456就充当了token的角色。

到这，仓库这边的配置就完成了，接下来的问题就是服务器如何接收这个请求并重新部署hexo了。

### VPS相关配置
在hexo目录中新建webhook.js,内容如下：
{% codeblock %}
var http = require('http')
var exec = require('child_process').exec
http.createServer(function (req, res) {
#该路径与WebHooks中的路径部分需要完全匹配，实现简易的授权认证。
if(req.url === '/webhooks/push/123456'){
#如果url匹配，表示认证通过，则执行 sh ./deploy.sh
exec('sh ./deploy.sh')
}
res.end()
}).listen(4002)
{% endcodeblock %}
这段代码就能启动一个nodejs服务，监听4002端口。
当请求过来的url完全匹配的时候，执行deploy.sh。
再新建一个文件deploy.sh处理部署相关脚本，内容如下：
{% codeblock %}
git pull origin master
{% endcodeblock %}
然后在服务器中启动nodejs服务监听webhooks
使用PM2执行脚本[PM2](http://www.cnblogs.com/zhongweiv/p/pm2.html)
{% codeblock %}
npm install pm2 -g
pm2 start webhook.js
{% endcodeblock %}
然后可以在本机中hexo d 命令，vps就会自动更新hexo文件了...