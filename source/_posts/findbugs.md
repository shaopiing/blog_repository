---
title: "知识百科系列 (一) —— FindBugs"
date: 2016-05-06 23:39:50
category: 效率插件
tags: [Program,WikiKnowledge]
---
在使用 Jenkins 构建 Java Web 项目时候，有一项叫做静态代码检查，是用内置的 findBugs 插件，对程序源代码进行检查，以分析程序行为的技术，应用于程序的正确性检查、安全缺陷检测、程序优化等，特点就是不执行程序。它有助于在项目早期发现以下问题：变量声明了但未使用、变量类型不匹配、变量在使用前未定义、不可达代码、死循环、数组越界、内存泄漏等。分为以下几种类型：

<!--more-->

**一、Bad Practice (糟糕的写法)**

**二、Correctness (不太的当)**

**三、Experimental (实验)**

**四、Internationalization (国际化)**

**五、Malicious code vulnerability (有漏洞的代码)**

**六、Multithreaded correctness (多线程问题)**

**七、Performance (执行)**

**八、Security (安全性)**

**九、Dodgy code (可疑代码)**

具体描述，可以参加如下地址：[FindBugs问题列表以及描述](http://findbugs.sourceforge.net/bugDescriptions.html)

常见的比如：

SBSC: Method concatenates strings using + in a loop (SBSC_USE_STRINGBUFFER_CONCATENATION)

问题描述已经很清楚了，尽量不要在循环中使用 String，用 StringBuffer 来代替：

The method seems to be building a String using concatenation in a loop. In each iteration, the String is converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a cost quadratic in the number of iterations, as the growing string is recopied in each iteration.

Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.

For example:

```java
  // This is bad

  String s = "";

  for (int i = 0; i < field.length; ++i) {

    s = s + field[i];

  }

  // This is better

  StringBuffer buf = new StringBuffer();

  for (int i = 0; i < field.length; ++i) {

  	buf.append(field[i]);

  }

  String s = buf.toString();

```

写段代码比较下：

```java
Long preSecond = System.currentTimeMillis();

String str = "";

int length = 10000;

for (int i = 0; i < length; i++) {

　　str += i;

}

System.out.println("cost " + (System.currentTimeMillis() - preSecond) + " seconds.");

Long posSecond = System.currentTimeMillis();

StringBuffer buffer = new StringBuffer();

for (int i = 0; i < length; i++) {

　　buffer.append(i);

}

System.out.println("cost " + (System.currentTimeMillis() - posSecond) + " seconds.");
```



输出结果为：

```java
cost 363 seconds.

cost 3 seconds.
```

**还有个错误关于实体类的 setter 和 getter 方法的：**

EI2: May expose internal representation by incorporating reference to mutable object (EI_EXPOSE_REP2)

This code stores a reference to an externally mutable object into the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Storing a copy of the object is better approach in many situations.

报的是这种比如 Date 类型的字段的 getter 和 setter 方法：

![](http://p8bc1hri5.bkt.clouddn.com/findBugs1.png)

这里的警告意思是，在进行 get 或者 set 时候，因为 Java 是引用传递，对象之间赋值，可能会导致其他对象的修改，所以建议的做法是，把对象的克隆对象赋值给需要赋值的对象。

首先，该实体类要继承 Cloneable 接口，然后在对应的 getter 和 setter 方法更改如下即可：

![](http://p8bc1hri5.bkt.clouddn.com/findBugs2.png)

在一款优秀的 Java IDE —— IntellijIDEA 中，也可以安装对应的插件，来将这些问题扼杀在项目上线之前，避免不必要的麻烦。

![](http://p8bc1hri5.bkt.clouddn.com/findBugs3.png)

安装以后，右击要分析的 Java 文件，选择 Analyzed Files 即可

![](http://p8bc1hri5.bkt.clouddn.com/findBugs4.png)

分析之后，如果有 bugs，就会显示，然后根据提示来修正即可

![](http://p8bc1hri5.bkt.clouddn.com/findBugs5.png)
