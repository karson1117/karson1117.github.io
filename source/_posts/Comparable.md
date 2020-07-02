---
title: Comparable 接口
date: 2017-03-15 16:37:39
tags:
	- Java基础
categories:
	- 编程
	- Java

---
![](/img/articleImg/compare.png)

>compareTo()方法的工作原理是返回一个int值——或正，或负，或为零。它通过调用作为参数的对象来比较对象。负数表示调用的对象比参数“轻”。如果我们用大小来比较苹果，那么上面的调用会返回一个负数，例如-400，因为红苹果比青苹果小。如果两个苹果重量相等，那么调用将返回0。如果红苹果更重，那么compareTo()将返回一个正数，例如68。
<!--more-->

### 例1：通过重量排序苹果
在第一个例子中，我们将通过重量对苹果排序。只需要一行代码。

	Collections.sort(apples);
上面的代码行可以为我们做到所有的排序工作，只要我们事先定义好如何对苹果进行排序（这就需要多行代码了）。
让我们开始写苹果类吧。
{% codeblock %}
public class Apple implements Comparable {
    private String variety;
    private Color color;
    private int weight;
    @Override
    public int compareTo(Apple other) {
        if (this.weight < other.weight) {
            return -1;
        }
        if (this.weight == other.weight) {
            return 0;
        }
        return 1;
    }
}
{% endcodeblock %}
这是Apple类的第一个版本。由于我们使用的是compareTo方法，并且正在排序苹果，所以我实现了Comparable接口。在这第一个版本中，我们通过重量比较对象。在我们的compareTo()方法中，我们写一个if条件，说明如果这个苹果的重量小于其他的苹果，那么返回一个负数，为了保持简单，我们假定它为-1。请记住，这意味着这个苹果轻于Apple ‘other’。在第二个if语句中，我们要说明，如果苹果重量相等，那么返回一个0。当然，如果这个苹果既不是更轻，又不是一样重，那就只能比其他苹果更重了。在这种情况下，我们返回一个正数，假定为1。

### 例2：通过多个特征排序苹果
正如我前面提到的，我们还可以使用compareTo()比较多个特征。比方说，我们第一通过品种排序苹果，但如果两个苹果是同一品种，那么我们就按颜色排序。最后，如果这两个特性相同，那么我们将按重量排序。虽然我们可以手动实现这件事，就像我在最后一个例子中做的那样，但是其实可以用一种简洁得多的方式实现。一般来说，最好是重用现有的代码，而不是自己写。我们可以在Integer、String和枚举类中使用compareTo方法来比较值。由于我们没有使用Integer对象，用了int，所以我们不得不使用来自于Integer包装器类的一个静态的helper方法来比较两个值。
{% codeblock %}
public class Apple implements Comparable {
    private String variety;
    private Color color;
    private int weight;
    @Override
    public int compareTo(Apple other) {
        int result = this.variety.compareTo(other.variety);
        if (result != 0) {
            return result;
        }
        if (result == 0) {
            result = this.color.compareTo(other.color);
        }
        if (result != 0) {
            return result;
        }
        if (result == 0) {
            result = Integer.compare(this.weight, other.weight);
        }
        return result;
    }
}
{% endcodeblock %}

在上例中，我们比较了客户指定的苹果的第一特性，它们的品种。如果compareTo()调用的结果为非零，那么我们返回值。否则，我们调用另一个compareTo()直到得到一个非零值，或者直到已经比较完这三个特征。尽管此代码可以工作，但它不是最有效或干净的解决方案。在例3中，我们重构我们的代码，使其更简单。
{% codeblock %}
@Override
public int compareTo(Apple other) {
     int result = this.variety.compareTo(other.variety);
     if (result == 0) {
          result = this.color.compareTo(other.color);
     }
     if (result == 0) {
          result = Integer.compare(this.weight, other.weight);
     }
     return result;
}
{% endcodeblock %}
正如你所看到的，这大大减少了代码，并且每一次比较只要一行代码。如果一个compareTo()调用的结果是零，那么我们就转移到下一个相同if语句的比较中。顺便说一句，这是成为Clean Coder的一个很好的例子。通常情况下，你不需要立即写出干净的代码；你可以从一个粗略的想法开始，使其可以工作，然后不断改进，直到你尽可能得让它干净就可以了。

### Comparable，hashCode以及Equals
你可能会注意到compareTo()看起来有点像hashCode()和equals()方法。但是，它们有一个重要的区别。对于hashCode()和equals()方法，比较个体属性的顺序不影响返回的值，但是，在compareTo()中，通过你比较对象的顺序来定义对象的顺序。

### 结论
在结论中我只想强调Comparable接口是多么的重要。它既用于java.util.Arrays，也用于java.util.Collections实用程序类，来排序元素和搜索排序集合中的元素。使用TreeSet和Tree Map，就更简单了——想要它们会自动排序必须实现Comparable接口的元素。

