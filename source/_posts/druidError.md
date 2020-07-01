---
title: connection holder is null
date: 2017-03-14 10:58:24
tags:
	- java
	- errors
categories:
	- 其他
---
>"Druid提供的getConnection()或者getConnection(long maxWaitMillis)方法不能保证>在同一个线程中获取的始终是一个连接，直到显式的将连接关闭吗？"。
>必须在程序在缓存从Druid中取出的连接才能保证现一个事务在使用的是同一个连接。 
>而抛出“connection holder is null”异常的原因可能在于： 
>removeAbandonedTimeout //关闭长时间不使用的连接超时时间,单位秒,默认30*1000

假设这个参数的值为30分钟,当一个连接在获取后30分钟还没释放,也就是Connection的DruidPooledPreparedStatement对象执行完了executXXX()方法但还未执行close、commit、rollback方法,对应于Connection的running参数的值为false,这时Durid的DestroyConnectionThread线程会自动将该连接回收。当程序要commit()连接时会执行checkState()方法,这个方法会执行以下代码：
{% codeblock %}
if(holder ==null)
    {
        if (disableError != null) {
            throw new SQLException("connection holder is null", disableError);
        } else {
            throw new SQLException("connection holder is null");
        }
    }
{% endcodeblock %}
这段代码就是我们看到的“connection holder is null”异常的来源，因此，我们需要做的就是根据Druid提供的监控信息（主要看“连接持有时间分布”的值）修改这个参数的值，它的值一定要比最长的连接持有时间还要大。