---
title: Lambda
date: 2017-03-13 15:23:43
tags:
	- java
	- Lambda
categories:
	- Java基础
---

Java8主要的改变是为集合框架增加了流的概念，提高了集合的抽象层次。相比于旧有框架直接操作数据的内部处理方式，流+高阶函数的外部处理方式对数据封装更好。同时流的概念使得对并发编程支持更强。
在语法上Java8提供了Lambda表达式来传递方法体,简化了之前方法必须藏身在不必要的类中的繁琐。Lambda表达式体现了函数式编程的思想，即一个函数亦可以作为另一个函数参数和返回值，使用了函数作参数/返回值的函数被称为高阶函数。

## 1.Lambda表达式
Java 被诟病为繁琐的地方就在于不支持传递方法，Java中的方法必须依赖类存在，也不能将方法作为参数或返回值，这是与python等语言相比的弱势。
Java 8中使用新特性Lambda表达式来改善这一点。
<!--more-->
### 1.1 使用示例
以Runnable接口为例，如果需要执行一个线程，实际只需要run()方法中的代码块，但形式上必须要先制造一个Runnable接口实现类(通常是匿名内部类)。
使用Lambda表达式仅仅需要一行代码，达到传递run方法的效果,而不必定义匿名内部类。
{% codeblock %}
new Thread(()->System.out.println("Lambda")).start();
{% endcodeblock %}
### 1.2 类型参数推断机制(Type Argument Inference)
Lambda表达式之所以能够做如此简化得益于Java的类型参数推断机制。所有省略的内容都可以由编译器通过上下文推断出来。类型推断机制在Java中的应用广泛，例如数组类型确定，Java7引入的菱形操作符等。类型参数推断机制要推断的是Lambda表达式的目标类型，往往需要与Java的重载解析机制配合。其解析规则是只有一个可能目标类型时，由响应函数接口里的参数类型推导得出有多个可能目标类型，选择最具体的类型有多个可能目标类型但无法明确最具体类型，则编译报错。

### 1.3 函数接口(Functional Interface)
一个方法可以抽象成函数接口。函数接口类似于一个黑箱，只需要关注其参数和返回值类型，函数接口中只有单方法。
Runnable的函数接口如下:
![](/img/articleImg/jk82.png)
可以看到这是一个空接口。可以用它代表所有参数和返回值都为空的方法。
Java8中定义若干函数接口(位于包java.util.function)。
![](/img/articleImg/jk8.png)
以Pridicate函数接口为例，这是一个泛型接口，参数可以是任意类型，返回值是boolean类型，代表根据数值作判断的一类方法。

### 1.4 并非语法糖
从类型推断的角度看很容易觉得Lambda表达式是和泛型，装箱等机制一样的语法糖，编译器在背后补全了省略信息，但实际上并非如此。
{% codeblock %}
class Apple{
	public String toString() {return "apple";};
	Runnable r1 = ()->{System.out.println(this);};
	Runnable r2 = new Runnable() {
		public void run() {
			System.out.println(this);
		}
	};
}
//执行两个线程得到的结果是
apple
Day0917.Apple$1@22e90474
{% endcodeblock %}

	正常的匿名内部类中 this关键字 指向内部类对象自身，同时将生成Apple$1.class文件。
	Lambda表达式中this所指向的则是外部类对象，并不会生成内部类class文件，这说明Lambda表达式并不是语法糖，它没有产生一个内部类，也没有引入一个新的作用域。
	Lambda与内部类相同之处在于其内部所定义的变量均为final或既成事实上的final.

### 1.5 默认方法
Java8最重要的改变就是对类库的改造，使得接口中方法可以拥有代码体。这种定义在接口中的包含方法体的方法，需要用default修饰，称之为默认方法。
{% codeblock %}
interface Apple{
	default void show(){
		System.out.println("interface");
	}
}
class MyApple implements Apple{
	@Override
	public void show() {
		Apple.super.show();
	}
}
{% endcodeblock %}
如果实现类中重写了默认方法，则接口中默认方法就被覆盖了。如果两个接口定义了相同的默认方法，则实现类中可以通过指定全称来确定使用哪个父类的方法。

### 1.6 方法引用
如果将匿名内部类改造为Lambda表达式是偷懒的话，那方法引用则是懒到连Lambda表达式都不想写了。
在之前，我们知道Lambda表达式可以作为函数参数和返回值，表示传递一个方法。方法引用就是使用 ClassName::MethodName 的形式来指定方法。故而方法引用与Lambda表达式完全同源同种，可以相互替代。

>//1,建立一个字符串
>String::new 
>//2.建立一个字符串数组
>String[]::new
>注意 lambda表达式与方法引用表示的是方法本身，将要被用过高阶函数的参数/返回值，并不能单独使用。

## 2.流stream
任务:创建一个姓名集合，要求出所有初始字母为a的人的总数目。使用流处理的代码如下:
{% codeblock %}
ArrayList<String> person = new ArrayList<>();
----init----
//1.由集合获得流对象
Stream<String> steam = person.stream();
//2.对流对象进行过滤和统计
steam.filter((s)->s.startsWith("a")) //1.流过滤
.count(); //2.计算流对象中元素数目
{% endcodeblock %}
使用函数接口(形式上表现为Lambda表达式)作为参数和返回值的函数就是所谓的高阶函数，如此处的filter，其参数为函数接口Predicate，亦可以理解为一个接口为 T--->boolean 的方法。
上述示例中为流对象的高阶函数传入一个函数接口Predicate，避免了直接处理集合中的数据对象。示例展示了流使用的通用格式:
获得流对象Stream
对流对象Stream进行惰性求值，返回值仍然是一个Stream对象。
对流对象Stream进行及早求值，返回值不在是一个Stream对象。

### 2.1常见高阶函数
#### 1.collect方法
collect方法属于一个及早求值方法，负责将流对象转换成其他数据结构，如列表，集合，值等。
这项工作由收集器Collector完成。java8为此提供了Collectors工具类。
#### 1.1 转换成集合
>List<Person> list = stream.collect(Collectors.toList());
>List<Person> arraylist = stream.collect(Collectors.toCollection(ArrayList::new));       
>Set<Person> set = stream.collect(Collectors.toSet());
>Set<Person> treeSet = stream.collect(Collectors.toCollection(TreeSet::new));

使用Collectors.toList()将流对象转换成集合时并不需要指定具体类型，Java默认选择了实现类型，如果要自己指定，可以使用Collectors.toCollection(ArrayList::new)，其参数ArrayList::new就是上文中的方法引用，表示一个建立ArrayList对象的方法，ArrayList就是想要转换成的数据类型；
#### 1.2 转换成值
>//1.获得最大最小值
>Function<Person, Integer> getLevel = p->p.age; 
>Comparator<Person> comparator = Comparator.comparing(getLevel);
>stream.collect(Collectors.maxBy(comparator));
>stream.collect(Collectors.minBy(comparator));
>//2.获得平均值
>ToIntFunction<Person> getAverage = p->p.age;
>stream.collect(Collectors.averagingInt(getAverage));

#### 1.3 数据分块
将流对象按某种条件分成两部分
>Predicate<Person> isTang = p->p.country.equals(Country.Tang);
>stream.collect(Collectors.partitioningBy(isTang));

#### 1.4 数据分组
>Function<Person, Integer> country= p -> p.country.ordinal();
>stream.collect(Collectors.groupingBy(country));

分块和分组看似相同，但意义不同，分块使用判断作为方法，只能将流分成两块；分组则灵活的多。
#### 1.5 字符串
>stream.map(Person::getName).collect(Collectors.joining("/", "[", "]"));

#### 1.6 合并收集器
>stream.collect(Collectors.groupingBy(country,Collectors.counting()));

### 2.map
map是一个惰性求值方法。函数接口为Function<T, R>函数接口,负责将数据从一个类型转换为另一个类型；高阶函数map的作用就是将数据从一个流转换为另一个流。
### 3.filter
filter 是一个惰性求值方法。函数接口为Pridicate<T>,此方法负责对数据进行判断，filter高阶函数负责根据判断结果对流进行过滤。
### 4.flatMap系列
flatMap 是一个惰性求值方法。其参数亦为Function<T, R>,将多个流组合为一个流。
{% codeblock %}
//1.a1,a2是两个列表，map处理后仍是两个列表
Stream.of(a1,a2).map(s->s)

[1, 2, 3, 4]
[]

//2.flatMap将二者合并为一个流
Stream.of(a1,a2).map(s->s)
.flatMap(s->s.stream())
{% endcodeblock %}
1234
看源码可知，flatMap中函数接口Function的输出类型为Stream<R>。
### 5.max/min
属于一个及早求值方法。需要传入一个Comparator函数接口，Java8提供了Comparator.comparing方法获得该函数接口的实现，该静态方法是接口的静态方法，获得一个函数返回一个Comparator对象。
min(Comparator.comparing(s->s.toString()));
max/min的返回值是 Optional，代表一个或有或无的值，主要是用来取代万恶的null值；使用get方法可以获取其值。
### 6.reduce
属于一个及早求值方法。意为流数据的累加，有两个版本。
{% codeblock %}
//1.无初始值累加
T t = person.stream().reduce((a,b)->a+b);
//2.带初始值累加
Optional<T> t = person.stream().reduce("1",(a,b)->a+b);
{% endcodeblock %}
### 7.foreach
属于一个及早求值方法，用来遍历流对象。
总而言之，Java8中流对象的引入使得可以在更高的层次上对集合进行处理，使得抽象的方法和具体的行为逻辑分离开来，也加强了数据的封装性，另一个好处是对并发的支持更强，以后再补充。