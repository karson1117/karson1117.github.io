---
title: springmvc导出Excel问题记录
date: 2017-03-31 16:18:15
tags:
	- spring
	- Excel
categories:
	- Spring	
---
>SpringMVC中获取不到POST形式的参数
>这是form表单的enctype编码方式不同导致的enctype属性规定在发送到服务器之前,应该如何
>对表单数据进行编码。默认地，表单数据会编码为"application/x-www-form-urlencoded"。
>就是说，在发送到服务器之前,所有字符都会进行编码（空格转换为 "+" 加号，特殊符号转换为
>ASCII HEX 值）。如果使用GET，则强制使用application/x-www-form-urlencoded"方式。
>但代码里强制使用了multipart/form-data方式。

<!--more-->
所以spring mvc如果要接收 multipart/form-data 传输的数据，应该在spring上下文配置：
{% codeblock %}
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">       
</bean>
{% endcodeblock %}
并将commons-fileupload-1.3.2jar包引入到项目中
{% codeblock %}
<!-- POI导出Exl -->
<dependency>
   <groupId>org.apache.poi</groupId>
   <artifactId>poi-ooxml</artifactId>
   <version>3.9</version>
</dependency>
<dependency>
   <groupId>commons-fileupload</groupId>
   <artifactId>commons-fileupload</artifactId>
   <version>1.3.2</version>
</dependency>
{% endcodeblock %}
这样服务端就既可以接收multipart/form-data 传输的数据，也可以接收application/x-www-form-urlencoded传输的文本数据了。

注：此问题是在上一个项目[git地址](https://coding.net/u/letra/p/mvcdemo/git)的基础上新增了导出Ecxel功能是发现，在此记录一二。