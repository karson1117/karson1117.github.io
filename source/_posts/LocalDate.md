---
title: java8中如何处理时间
date: 2017-04-01 13:12:06
tags: 
	- java
categories:
	- Java基础
---

Java8之前，Date类都是可变类
当我们在多线程环境下使用它
编程人员应该确认Date对象的线程安全
Java8的Date和Time API提供了线程安全的不可变类
编程人员不用考虑并发的问题
![](/img/articleImg/time2.jpg)
<!--more-->

### LocalDate用法
LocalDate只提供日期不提供时间信息。它是不可变类且线程安全的
{% codeblock %}
// 取当前日期：
LocalDate today = LocalDate.now(); // -> 2017-04-01
// 根据年月日取日期，04月就是04：
LocalDate crischristmas = LocalDate.of(2017, 04, 01); // -> 2017-04-01
// 根据字符串取：
LocalDate endOfFeb = LocalDate.parse("2017-04-01"); 
// 严格按照ISO yyyy-MM-dd验证，04写成4都不行，当然也有一个重载方法允许自己定义格式
LocalDate.parse("2017-02-29"); // 无效日期无法通过：DateTimeParseException: Invalid date...
{% endcodeblock %}
日期转换经常遇到，比如：
{% codeblock %}
// 取本月第1天：
LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth()); // 2017-04-01
// 取本月第2天：
LocalDate secondDayOfThisMonth = today.withDayOfMonth(2); // 2017-04-02
// 取本月最后一天，再也不用计算是28，29，30还是31：
LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth()); // 2017-04-30
// 取下一天：
LocalDate nextDayOf = lastDayOfThisMonth.plusDays(1); // 变成了2017-05-01
// 取2017年1月第一个周一，这个计算用Calendar要死掉很多脑细胞：
LocalDate firstMondayOf2017 = LocalDate.parse("2017-01-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 2017-01-02
{% endcodeblock %}
### LocalTime
LocalTime只提供时间而不提供日期信息，它是不可变类且线程安全的
{% codeblock %}
LocalTime now = LocalTime.now(); // 11:09:09.240
清除毫秒数：
LocalTime now = LocalTime.now().withNano(0)); // 11:09:09
构造时间：
LocalTime zero = LocalTime.of(0, 0, 0); // 00:00:00
LocalTime mid = LocalTime.parse("12:00:00"); // 12:00:00
{% endcodeblock %}
### JDBC

最新JDBC映射将把数据库的日期类型和Java 8的新类型关联起来：
>date -> LocalDate
time -> LocalTime
timestamp -> LocalDateTime

再也不会出现映射到java.util.Date其中日期或时间某些部分为0的情况了.