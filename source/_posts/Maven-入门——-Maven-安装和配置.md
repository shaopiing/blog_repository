---
title: Maven 入门—— Maven 安装和配置
date: 2018-08-01 23:16:56
tags: "Program"
---

## 二、Maven 的安装和配置
### 1、安装
**Windows 环境**
- Maven 官网下载安装文件
- 解压到指定目录
- 配置环境变量（M2\_HOME）
- cmd 输入 `mvn -v`

**Mac环境**
- `brew install mvn`
- 配置环境变量 
- export `M2\_HOME=/usr/local/Cellar/maven/3.5.4`，
- export `PATH=$PATH:$M2\_HOME/bin`
- 终端输入：`mvn -v`
  ![](http://p8bc1hri5.bkt.clouddn.com/Maven%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE1.png)
### 2、配置
- Eclipse：**m2eclipse**
- IDEA：自带+辅助插件
  ![](http://p8bc1hri5.bkt.clouddn.com/Maven%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE2.png)
### 3、最佳实践
#### 1、设置 MAVEN\_OPTS 环境变量
运行 mvn 命令实际上是执行了 Java 命令，那么 Java 命令可用的参数同样可用在运行 mvn 命令时可用。
通常需要设置 MAVEN\_OPTS 的值为 -Xms128m -Xmx512m（堆内存的初始值和最大值），因为 Java 默认的最大可用内存往往不够满足 Maven 运行的需要，比如在项目较大时，使用 Maven 生成项目站点需要占用大量的内存，如果没有该配置，很容易得到 `java.lang.OutOfMemeoryError`，因此，最好提前配置该变量。
设置方式建议参考 **M2\_HOME** 变量的配置方式，不要直接更改安装目录下的文件，不然版本更新以后还要重新配置该变量。
#### 2、配置用户范围 settings.xml
Maven 用户可以选择配置 `$M2\_HOME/conf/settings.xml` 或者 `~/.m2/settings.xml`，前者是全局范围的，后者是用户范围的，推荐使用用户范围的 settings.xml，主要是为了避免影响其他的用户，而且配置用户范围的 settings.xml 文件还便于 Maven 升级，升级时不会影响到 Maven 的安装文件，也不会影响到使用。因为使用有个加载顺序的，先加载用户的配置文件，没有匹配再加载系统的配置文件：
![](http://p8bc1hri5.bkt.clouddn.com/Maven%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE3.png)
#### 3、不用使用 IDE 内嵌的 Maven![image-20180720005349463](/var/folders/3g/9x_v6q3n61g8sffb0712wl3h0000gn/T/abnerworks.Typora/image-20180720005349463.png)

![markdown/20180720010533.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180720010533.png)

无论 Eclipse 还是 IDEA，当集成 Maven 时，都会安装上一个内嵌的 Maven，这个内嵌的 Maven 通常会比较新，但是不一定稳定，而且往往也会和在命令行使用的 Maven 不是同一个版本。这样就有可能因为版本不同的原因出现某些问题，所以建议还是用本地安装的 Maven 版本，而本地安装的版本也应该与服务器上安装的版本一致。
![](http://p8bc1hri5.bkt.clouddn.com/Maven%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE4.png)