---
title: Hexo博客多电脑同步（hexo+GitHub）
date: 2020-06-29 09:58:13
tags: 
- Hexo
- GitHub
- Git
---

### 1.如何让公司电脑A、家里电脑B都能同步编辑博客。

**上传博客工程到Github**

 首先在公司的A电脑搭建并部署完系统后，我们需要将项目上传到你的github上。在A电脑上执行如下命令:

```js
#git初始化
git init
#添加仓库地址
git remote add origin https://github.com/用户名/你的GitHub用户名.github.io.git 
#新建分支并切换到新建的分支
git checkout -b 分支名 
#添加所有本地文件到git
git add . 
#git提交
git commit -m "这里填写你本次提交的备注，内容随意" 
#文件推送到hexo分支
git push origin 分支名 
```

**从另一台电脑下载博客工程**

B电脑如何下载项目文件呢？首先在B电脑上部署好Git和Node.js环境。

然后输入以下命令<!--more-->

```js
git clone -b 分支名 https://github.com/用户名/你的GitHub用户

```

克隆下载完成后，进入到你项目的文件夹，重新配置你的hexo环境，命令如下：

```js
#安装hexo,注意这里不需要hexo初始化,否则之前的hexo配置参数会重置
sudo npm install -g hexo-cli 
#安装依赖库
sudo npm install 
#安装git部署相关配置
sudo npm install hexo-deployer-git 
```

之后就可以创建撰写新的文章，并使用sudo hexo g -d命令创建并部署您的网站。

**撰写完后如何再次同步**

```js
git add .
git commit -m "提交的备注，内容随意"
git push origin 分支名
#没错，这个样就够了~你B电脑上的数据也已经同步到Github上面了。
#那第二天到A电脑跟前，只需要执行以下命令就行

git pull
#这样，你的数据就全部同步到A电脑了，以后在部署完后，再次执行

git add .
git commit -m "提交的备注，内容随意"
git push origin 分支名
```

------

**常见问题**

（1）修改主题后，主题文件无法推送至GitHub

可能是该子文件夹下有.git文件夹导致无法上传，

```markdown
#删除子文件夹下.git后，依然无法提交子文件夹下的文件。
1. git rm --cached themes/yilia
2. git add .
3. git commit -m "xxx"
4. git push origin master
```

（2）文章多标签格式：

```js
tags: 
- Hexo
- GitHub
- Git
```

（3）文章缩略标识

```
<!--more-->
```



（4）添加评论GitTalk

**创建 gitalk.ejs**

在你的 hexo 目录 `/theme/yilia/layout/_partial/post/` 目录下创建 `gitalk.ejs` 并写入如下内容：

```
<div id="gitalk-container"></div>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.js"></script>

<script>
var gitalk = new Gitalk({
  clientID: '<%=theme.gitalk.clientID%>',
  clientSecret: '<%=theme.gitalk.clientSecret%>',
  repo: '<%=theme.gitalk.repo%>',
  owner: '<%=theme.gitalk.owner%>',
  admin: ['<%=theme.gitalk.admin%>'],
  id: md5(window.location.pathname),
  distractionFreeMode: <%=theme.gitalk.distractionFreeMode%>
})

gitalk.render('gitalk-container')
</script>
```

**修改 article.ejs**

在你的 hexo 目录 `/theme/yilia/layout/_partial/article.ejs` 文件中最后一行 `“<% } %>”` 之前添加如下内容：

```
<% if(theme.gitalk.enable && theme.gitalk.distractionFreeMode){ %>
      <%- partial('post/gitalk', {
      key: post.slug,
      title: post.title,
      url: config.url+url_for(post.path)
    }) %>
  <% } %>
```

**添加配置文件**

在 yilia 的配置文件`_config.yml` 中 gitment 配置下面添加如下配置文件

```
#6. Gitalk
gitalk: 
  enable: true    #用来做启用判断可以不用
  clientID: your clientID    #Github上生成的 Settings Developer/settings/OAuth Apps
  clientSecret: your clientSecret   #同上
  repo: git_comment    #评论所在的github project
  owner: findtheonlyway    #github用户名
  admin: erbiduo    #可以初始化评论issue的github账户名称
  distractionFreeMode: true
```



（4）微信分享二维码失效

打开`themes\yilia\layout\_partial\post\share.ejs`文件

把第49行中的 `//pan.baidu.com/share/qrcode?url=`修改为：

```js
//api.qrserver.com/v1/create-qr-code/?size=150x150&data=
```

[参考链接](https://cloud.tencent.com/developer/article/1046404)