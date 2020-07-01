---
title: java值传递还是引用传递
date: 2017-05-24 15:48:36
tags:
	- java
categories:
	- Java基础	
---

 由一道面试题引发的问题：java是值传递还是引用传递？
{% codeblock %}
public class Test {

    String str="abc";
    int a[]={1,2,3};
    int i=1;
    void change(String str,int a[],int i){
        
        str="cbd";
        a[0]=4;
        i=2;
    }
    public static void main(String[] args) {
    
        Test t=new Test();
        t.change(t.str,t.a,t.i);
        System.out.println(t.str);
        System.out.println(t.a[0]);
        System.out.println(t.i++);
    }
}
{% endcodeblock %}

<!--more-->
>运行结果
>abc
>4
>1

### 1.按值传递是什么
指的是在方法调用时，传递的参数是按值的拷贝传递。
按值传递重要特点:传递的是值的拷贝，也就是说传递后就互不相关了。

### 2.按引用传递是什么
指的是在方法调用时，传递的参数是按引用进行传递，其实传递的引用的地址，也就是变量所对应的内存空间的地址。
按引用传递的重要特点:
传递的是值的引用，也就是说传递前和传递后都指向同一个引用(也就是同一个内存空间)。

详细博文见[>>>](http://blog.csdn.net/zzp_403184692/article/details/8184751)
