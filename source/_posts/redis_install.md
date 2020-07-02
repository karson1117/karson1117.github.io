---
title: Centos6下redis安装配置
date: 2017-07-21 13:34:00
tags:
	- Redis
categories:
	- 运维
---
>Remote Dictionary Server(Redis)
>是一个由Salvatore Sanfilippo写的key-value存储系统。Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
>
>它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map),列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

<!--more-->

### 1、安装需要的支持环境

在安装Redis之前首要先做的是安装Unix的Tcl工具，如果不安装的话后期将无法对Redis进行测试。在后期执行make test的时候返回如下错误信息：You need tcl 8.xuyao de5 or newer in order to run the Redis test，具体的流程为：
{% codeblock %}
cd /usr/local/src
wget http://downloads.sourceforge.net/tcl/tcl8.6.3-src.tar.gz
tar -zxvf tcl8.6.3-src.tar.gz
cd ​tcl8.6.3/unix/
./configure
make
make install
{% endcodeblock %}
### 2、安装redis
安装redis的过程非常的简单，具体教程官网也有。具体如下：http://redis.io/download
{% codeblock %}
cd /usr/local/src
wget http://download.redis.io/releases/redis-4.0.0.tar.gz
tar zxvf redis-2.8.19.tar.gz
cd redis-2.8.19
make
make PREFIX=/usr/local/redis install
{% endcodeblock %}
其中PREFIX=/usr/local/redis可以省略，省略情况下redis会默认安装到/usr/local/bin目录下。
### 3、测试Redis
	cd src
	make test
	通过以上命令就可以对redis进行加大的测试。
### 4、配置redis
{% codeblock %}
#拷贝并修改配置文档
cp ./redis.conf /usr/local/redis/
vim /usr/local/redis/redis.conf
{% endcodeblock %}
{% codeblock %}
#我只修改了如下几项：
daemonize yes #redis将以守护进程的方式运行，默认为no会暂用你的终端
timeout 300​ #当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
requirepass yourpassword #设置密码
#bind 127.0.0.1 #限制了只能本地连接
另外，设置密码后，使用redis-cli登录要带密码登录
否则操作redis会出现身份认证的错误
命令如下:
redis-cli -a youPassword
{% endcodeblock %}
{% codeblock %}
#B、启动或关闭服务
service redis start
service redis stop
{% endcodeblock %}
### 5、使用redis
{% codeblock %}
[root@localhost redis]# cd /usr/local/redis/bin
[root@localhost bin]# ./redis-cli
127.0.0.1:6379> set name cjs
OK
127.0.0.1:6379> get name
"cjs"
127.0.0.1:6379>
{% endcodeblock %}
### 6、java使用redis

{% codeblock %}
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
{% endcodeblock %}
	public final class RedisUtil {

    //Redis服务器IP
    private static String ADDR = "***.***.***.***";
    
    //Redis的端口号
    private static int PORT = 6379;
    
    //访问密码
    private static String AUTH = "****";
    
    //可用连接实例的最大数目，默认值为8；
    //如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
    private static int MAX_ACTIVE = 1024;
    
    //控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8。
    private static int MAX_IDLE = 200;
    
    //等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException；
    private static int MAX_WAIT = 10000;
    
    private static int TIMEOUT = 10000;
    
    //在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的；
    private static boolean TEST_ON_BORROW = true;
    
    private static JedisPool jedisPool = null;
    
    /**
     * 初始化Redis连接池
     */
    static {
        try {
            JedisPoolConfig config = new JedisPoolConfig();
    		config.setMaxIdle(MAX_IDLE);
    		//jedis高版本JedisPoolConfig没有maxActive改名为：
            config.setMaxTotal(MAX_ACTIVE);
            config.setMaxWaitMillis(MAX_WAIT);
            config.setTestOnBorrow(TEST_ON_BORROW);
            jedisPool = new JedisPool(config, ADDR, PORT, TIMEOUT, AUTH);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 获取Jedis实例
     * @return
     */
    public synchronized static Jedis getJedis() {
        try {
            if (jedisPool != null) {
                Jedis resource = jedisPool.getResource();
                return resource;
            } else {
                return null;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
    
    /**
     * 释放jedis资源
     * @param jedis
     */
    public static void returnResource(final Jedis jedis) {
        if (jedis != null) {
            jedisPool.returnResource(jedis);
        }
    }
    }
