---
title: "设计模式系列 (一) —— 面向对象设计原则"
date: 2017-07-02 00:04:13
category: 程序人生
tags: [Program,设计模式]
---
### 1、单一职责原则(Single Responsibility Principle，SRP)

> 定义：一个类只负责一个功能领域中相应职责

单一原则告诉我们：一个类不能太“累”！承担的职责越多，被复用的可能性越小，耦合度越高，
所以，`单一职责原则是实现高内聚、低耦合的指导方针，是最简单又最难用的原则`。

### 2、开闭原则(Open-Closed Principle，OCP)

> 定义：软件实体应对扩展开放，而对修改关闭

### 3、里氏替换原则(Liskov Substitution Principle，LSP)

> 定义：所有引用基类对象的地方能够透明地使用其子类的对象

### 4、依赖倒转原则(Dependence Inversion Principle，DIP)

> 定义：抽象不应该依赖于细节，细节应该依赖于抽象

### 5、接口隔离原则(Interface Segregation Principle，ISP)

> 定义：使用多个专门的接口，而不使用单一的总接口

### 6、合成复用原则(Composite Reuse Principle，CRP)

> 定义：尽量使用对象组合，而不是继承来达到复用的目的

### 7、迪米特原则(Law of Demeter，LoD)

> 定义：一个软件实体应当尽可能少地与其他实体发生相互作用