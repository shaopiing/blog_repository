---
title: "疑难杂症处理系列 (二) —— Mac 下配置 Intellij IDEA ＋ Tomcat 出现权限问题的解决办法"
date: 2018-03-15 23:29:13
category: 程序人生
tags: [Program,ProblemFixed]
---

### 缘起 

今天构建 web 项目时启动 tomcat 服务，出现如下错误：

```java
Error running Tomcat 8.0.18: Cannot run program 

"/Users/emma/Desktop/tools/apache-tomcat-8.5.24/bin/catalina.sh" 

(in directory "/Users/emma/Desktop/tools/apache-tomcat-8.5.24/bin"): 

error=13, Permission denied
```

提示的问题主要是 **权限不足**，查阅资料，找到了解决办法：
打开终端，进入 tomcat\bin 目录，然后执行 chmod 777 *.sh，再次启动，果然好了。。。

本着刨根问底的态度，再次查阅了这个命令的含义，原来是在 Unix 和 Linux 各种操作系统下，设定文件权限（读、写、运行）的另一种写法。

例如我用 ls -l 命令列文件时，得到如下输出：
-rw-r--r--  1 emma  staff  2452  1 14 11:54 pom.xml
从第二个字符起 rw- 表示用户 emma 有读、写的权限，没有运行权限，接着的 r-- 表示用户组 staff 只有读的权限，没有其他权限，最后的 r-- 指其他人只有读的权限，没有写和运行的权限，这个是系统的默认设置。
读、写、运行三项权限可以用数字表示，就是 r=4，w=2，x=1。所以，上面的例子中的 rw-r--r-- 用数字表示成 644。反过来说 777 就是 rwxrwxrwx，意思就是该登录用户、他所在的组合其他人都有最高权限。

### 使用指令 chmod

使用权限 : 所有使用者
使用方式 : chmod [-cfvR] [--help] [--version] mode file...
说明 : Linux/Unix 的档案存取权限分为三级 : 档案拥有者、群组、其他。利用 chmod 可以藉以控制档案如何被他人所存取。

　　参数格式 :

　　mode : 权限设定字串，格式如下 : [ugoa...][[+-=][rwxX]...][,...]，其中

u：表示该档案的拥有者;

g：表示与该档案的拥有者属于同一个群体 (group) 者;

o：表示其他以外的人，a 表示这三者皆是;

+：表示增加权限、- 表示取消权限、= 表示唯一设定权限;

r：表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该档案是个子目录或者该档案已经被设定过为可执行;

-c：若该档案权限确实已经更改，才显示其更改动作

-f：若该档案权限无法被更改也不要显示错误讯息

-v：显示权限变更的详细资料

-R：对目前目录下的所有档案与子目录进行相同的权限变更 (即以递回的方式逐个变更)

--help：显示辅助说明

--version：显示版本

范例 : 

将档案 file1.txt 设为所有人皆可读取 :
`chmod ugo+r file1.txt`

将档案 file1.txt 设为所有人皆可读取 :
`chmod a+r file1.txt`

将档案 file1.txt 与 file2.txt 设为该档案拥有者，与其所属同一个群体者可写入，但其他以外的人则不可写入 :
`chmod ug+w,o-w file1.txt file2.txt`

将 ex1.py 设定为只有该档案拥有者可以执行 :
`chmod u+x ex1.py`

将目前目录下的所有档案与子目录皆设为任何人可读取 :
`chmod -R a+r *`

此外 chmod 也可以用数字来表示权限如 chmod 777 file

语法为：`chmod abc file`

其中 a,b,c 各为一个数字，分别表示 User、Group、及 Other 的权限。
r=4，w=2，x=1
若要 rwx 属性则 4+2+1=7；
若要 rw - 属性则 4+2=6；
若要 r-x 属性则 4+1=7。

范例：

`chmod a=rwx file` 和 `chmod 777 file` 效果相同

`chmod ug=rwx,o=x file` 和 `chmod 771 file` 效果相同



#### 相关链接：
- <a href="http://www.jianshu.com/p/q81RER" target="_blank">http://blog.csdn.net/ch717828/article/details/48554845</a>
- <a href="http://wowubuntu.com/markdown/" target="_blank">https://www.cnblogs.com/sipher/articles/2429772.html</a>