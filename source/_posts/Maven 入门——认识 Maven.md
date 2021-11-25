---
title: Maven 入门——认识 Maven
date: 2018-09-15 03:17:02
tags: "Program"
---

## 认识 Maven

Maven /ˈmāvən/ ，可以翻译成“专家”，是一款来自 Apache 组织的开源项目，用于项目管理。主要服务于基于 Java 平台的项目构建、依赖管理和项目信息管理。
构建（build）是每一位程序员每天都在做的事情，每天都有相当一部分时间花在编译、运行单元测试、生成文档、打包和部署等工作上，这些就是构建。而这一切，Maven 都可以帮我们自动完成，提高工作效率。

### 1、Ant vs Maven
Ant 只能算作一个构建工具，而 Maven 是一个项目管理工具，更正式的定义为：Maven 是一个项目管理工具，它包含了一个项目对象模型 (Project Object Model)，一组标准集合，一个项目生命周期 (Project Lifecycle)，一个依赖管理系统 (Dependency Management System)，和用来运行定义在生命周期阶段 (phase) 中插件 (plugin) 目标 (goal) 的逻辑。（Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.）

Apache Ant
1，Ant 没有正式的约定如一个一般项目的目录结构，你必须明确的告诉 Ant 哪里去找源代码，哪里放置输出。随着时间的推移，非正式的约定出现了，但是它们还没有在产品中模式化。
2，Ant 是程序化的，你必须明确的告诉 Ant 做什么，什么时候做。你必须告诉它去编译，然后复制，然后压缩。
3，Ant 没有生命周期，你必须定义目标和目标之间的依赖。你必须手工为每个目标附上一个任务序列。

```xml
<?xml version="1.0"?>

<project name="Hello" default="compile">

	<target name="compile" description="compile the Java source code to class files">

		<mkdir dir="classes"/>

		<javac srcdir="." destdir="classes"/>

	</target>

	<target name="jar" depends="compile" description="create a Jar file">

		<jar dstfile="hello.jar">

			<fileset dir="classes" includes="* * / *.class"/>

			<mainfest>

				<attribute name="Main-Class" value="HelloProgram"/>

			</mainfest>

		</jar>

	</target>

</project>
```



Apache Maven
1，Maven 拥有约定，因为你遵循了约定，它已经知道你的源代码在哪里。它把字节码放到  target/classes ，然后在  target 生成一个 JAR 文件。
2，Maven 是声明式的。你需要做的只是创建一个 pom.xml 文件然后将源代码放到默认的目录。Maven 会帮你处理其它的事情。
3，Maven 有一个生命周期，当你运行 mvn install 的时候被调用。这条命令告诉 Maven 执行一系列的有序的步骤，直到到达你指定的生命周期。遍历生命周期旅途中的一个影响就是，Maven 运行了许多默认的插件目标，这些目标完成了像编译和创建一个 JAR 文件这样的工作。

### 2、被误解的 Maven
C\++ 之父 Bjarne Stroustrup 说过一句话：“只有两类计算机语言，一类语言天天被人骂，还有一类没人用”。
用户最多的 Java 得到的骂声就不绝于耳，Maven 的用户也提出了很多质疑：

> “Maven 对于 IDE 的支持较差，bug 多，而且不稳定”

Maven 最高效的方式永远是命令行，IDE 在自动化构建方面有天生的缺陷。



> “Maven 采用了一种糟糕的插件系统来执行构建，新的、破损的插件会让你的构建莫名其妙地失败”

自 Maven 2.0.9 开始，所有核心的插件都设定了稳定版本，Maven 社区也提倡为你使用的任何插件设定稳定的版本，从 Maven 3 开始，如果你使用插件时未设定版本，会看到警告信息。



> “Maven 的仓库十分混乱，当无法从仓库中得到需要的类库时，我需要手动下载复制到本地仓库中”

Maven 的中央仓库确实不完美，你也许会发现某个 jar 包出现在两个不同的路径下，而这是开源项目本身改变了自身的坐标，假设一下没有中央仓库，你需要从开源项目首页寻找下载链接，这个反而是更痛苦的事情。



> “缺乏文档是理解和使用 Maven 的一个主要障碍”

这是事实， Maven 官方站点的文档十分凌乱，各种插件的文档更是需要费力寻找。