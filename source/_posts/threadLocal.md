---
title: ThreadLocal
date: 2020-07-28 15:56:16
categories:
	- 编程
	- Java
---

### ThreadLocal的用途

◆典型场景1:每个线程需要—个独享的对象（通常是工具类（线程不安全），典型需要使用的类有 SimpleDateFormat和 Random）

◆典型场景2:每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦。

实例1：SimpleDateFormat的格式化日期

（1）2个线程分别用自己的 SimpleDateFormat，这没问题。
（2）后来延伸出10个，那就有10个线程和10个SimpleDate Format，这虽然写法不优雅（应该复用对象）但勉强可以接受。
（3）但是当需求变成了1000个，那么必然要用线程池（否则消耗内存太多）。
（4）所有的线程都共用同一个 simple Date Format对象（加锁）。

<!--more-->


实例2:当前用户信息需要被线程内所有方法共享

◆一个比较繁琐的解决方案是把user作为参数层层传递，从 service-1传到 service-2，再从 service-2传到 Service-3，以此类推，但是这样做会导致代码冗余且不易维护。

◆每个线程内需要保存全局变量，可以让不同方法直接使用，避免参数传递的麻烦。

◆在此基础上可以演进，使用 UserMap。

◆当多线程同时工作时，我们需要保证线程安全，可以用 synchronized也可以用 ConcurrentHash Map，但无论用什么，都会对性能有所影响。

更好的办法是使用ThreadLocal，这样无需synchronized， 可以在不影响性能的情况下，也无需层传递参数，就可达到保存当前线程对应的用户信息的目的

◆用 ThreadLocal中保存一些业务内容（用户权限信息、从用户系统获取到的用户名、ID等）。

◆这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的。

◆在线程生命周期内，都通过这个静态 Threadlocal实例的get()方法取得自己set()过的那个对象，避免了将这个对象（例如user对象）作为参数传递的麻烦。

◆强调的是同一个请求内（同一个线程内）不同方法间的共享。

◆不需重写 initialValue0方法，但是必须手动调用set0方法。

### Threadloca的两个作用

1.让某个需要用到的对象在线程间隔离（每个线程都有自己的独立的对象）

2.在任何方法中都可以轻松获取到该对象

### ThreadLocal的好处

- 达到线程安全
- 不需要加锁，提高执行效率
- 更高效的利用内存、节省开销：相比于每个任务都新建一个SimpleDateFormat,显然用ThreadLocal可以节省内存和开销。
- 免去传参的繁琐：无论是场景一的工具类，还是场景二的用户名，都可以在任何地方直接通过ThreadLocal拿到，再也不需要每次都传同样的参数。ThreadLocal使得代码耦合度更低，更优雅

### 原理、源码分析

![原理图](/articleImage/2020-07-28/1.png)

搞清楚Thread,ThreadLocal以及ThreadLocalMap三者之间的关系

每个Thread对象中都持有一个ThreadLocalMap成员变量

#### 主要方法介绍

**initialValue()**
1.该方法会返回当前线程对应的“初始值”，这是一个延迟加载的方法，只有在调用get的时候，才会触发。
2.当线程第一次使用get方法访问变量时，将调用此方法，除非线程先前调用了set方法，在这种情况下，不会为线程调用本initialValue方法
3.通常，每个线程最多调用一次此方法，但如果已经调用了remove()后，再调用get()，则可以再次调用此方法
4.如果不重写本方法，这个方法默认返回null。一般使用匿名内部类的方法来重写initialValue()方法，以便在后续使用中可以初始化副本对象

**set()**

为这个线程设置一个新值

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
       map.set(this, value);
    else
        createMap(t, value);
}

```

**get()**

得到这个线程对应的value。
如果是首次调用get()，则会调用initialize来得到这个值；get方法是先取出当前线程的ThreadLocalMap，然后调用map.getEntry方法，把本ThreadLocal的引用作为参数传入，取出map中属于本ThreadLocal的Value

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
         }
    }
    return setInitialValue();
}

```

**remove**()

删除对应这个线程的值

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

ThreadLocalMap类，也就是Thread.threadLocals
ThreadLocalMap类是每个线程Thread类里面的变量，里面最重要的是一个键值对数组Entry[]table,可以任务是一个map，键值对：
    键：当前ThreadLocal
	值：实际需要的成员变量    

**ThreadLocalMap解决冲突的方式**

HashMap 处理冲突：拉链法-->红黑树

ThreadLocalMap这里采用的是线性探测法，也就是如果发生冲突，就继续找下一个空位置。

#### 内存泄漏问题

什么是内存泄漏：某个对象不再使用了，但是占用的内存却不能被回收

弱引用的特点是，如果这个对象只被弱引用关联（没有任何强引用关联），那么这个对象就可以被回收

弱引用不会阻止GC，因此这个弱引用的机制

```java
 static class Entry extends WeakReference<ThreadLocal<?>> {
       /** The value associated with this ThreadLocal. */
       Object value;
	   Entry(ThreadLocal<?> k, Object v) {
       super(k);
       value = v;
   }
}

```

Value的泄漏

（1）ThreadLocalMap的每个Entry都是一个对 Key的弱引用，同时每个Entry都包含了一个队value的强引用。

正常情况下，当线程终止，保存在ThreadLocal里的value会被垃圾回收，因为没有任何强引用了。但是，如果线程不终止（比如线程需要保持很久），那么key对应的value就不能被回收。因为value和Thread之间还存在这个强引用链路，所以导致value无法回收，就可能会出现OOM。

（2）JDK已经考虑到了这个问题，所以在set，remove,rehash方法中会扫描 key 为null的 Entry，并把对应的value设置为null，这样value对象就可以被回收

```java
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

     for (int j = 0; j < oldLen; ++j) {
          Entry e = oldTab[j];
          if (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null; // Help the GC
             } else {
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                  newTab[h] = e;
                 count++;
              }
          }
      }
     setThreshold(newLen);
     size = count;
     table = newTab;
}
```

（3）但是如果一个ThreadLocal不被使用，那么实际上set，remove，rehash方法也不会被调用，如果同时线程又不停止，呢么调用链就一直存在，那么就导致了value的内存泄漏。

（4）调用remove方法，就会删除对应的Entry对象，可以避免内存泄漏。所以使用完ThreadLocal后，应该主动调用remove()方法。

#### ThreadLocal注意的点

##### 空指针异常

若ThreadLocal 没有提前赋值，则返回null，此时应注意包装类的装箱拆箱会导致NPE （若用基本类型接收null即会NPE）

##### 共享对象

如果在每个线程中ThreadLocal.set()进去的东西本来就是多线程共享的同一个对象，比如static对象，那么多个线程的ThreadLocal.get()取得得还是这个共享对象本身，还是有并发访问问题。

**灵活运用**

如果可以不适用ThreadLocal就可以解决问题，那么不要强行使用：
例如在任务数很少的时候，在局部变量中可以新建对象就可以解决问题，那么就不需要使用到ThreadLocal。

优先使用框架的支持，而不是自己创造：
例如在Spring中，如果可以使用RequestContextHolder,那么就不需要自己维护ThreadLocal，因为自己可能会忘记调用remove()方法等，造成内存泄漏。

#### 实际应用场景--在Spring中的实例

DateTimeContextHolder

RequestContextHolder

*每次Http请求都对应一个线程，线程之间相互隔离，这就是ThreadLocal的典型应用场景*