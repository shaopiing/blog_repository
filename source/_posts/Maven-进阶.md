---
title: Maven 进阶
date: 2018-10-20 23:46:31
tags: "Program"
---

## 一、Maven 版本管理

Maven 的推荐版本号约定为：`主版本号.次版本号.增量版本号-<里程碑版本>`
开发中的版本要以 `-SNAPSHOT` 结尾，因为这种快照版本是支持 jar 包被覆盖的，那么，开发时候的 Maven 命令应该使用 `mvn clean package -U (强制拉一次)`
快照版本可以升级为正式版本的条件：
<!-- more -->
- 所以自动化测试应对全部通过

- 项目没有配置任何快照版本的依赖

- 项目没有配置任何快照版本的插件

- 项目所包含的代码已经全部提交到版本控制系统中

  

## 二、Maven 生命周期和常用命令
### compile
执行该命令会把代码进行编译
### clean
执行该命令会把 `/target` 目录下清空
### test
执行该命令会运行项目下的所有 test case
### package
执行该命令会对项目进行打包
### install
将 jar 包安装到本地仓库中，在多模块的项目中，如果依赖的模块发生变更，需要重新执行 install 才能生效
### deploy
把本地 jar 包发布到远端私服地址
### Maven 的生命周期
理解下边两句话，就理解了 Maven 的生命周期：
> A Build Lifecycle is Made Up of Phases.
> A Build Phase is Made Up of Plugin Goals.

![markdown/20180724234841.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180724234841.png)

![markdown/20180724234920.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180724234920.png)



## 三、Maven 常用插件

### 两个插件地址：
`https://maven.apache.org/plugins/`
`http://www.mojohaus.org/plugins.html`
### tomcat7-maven-plugin
### findbugs-maven-plugin
### maven-checkstyle-plugin
### maven-enforcer-plugin
### maven-source-plugin



## 四、Maven 自定义插件
### 自定义插件
新建一个 Maven 项目，将 pom.xml 里边的打包方式更改为 `<packaging>maven-plugin</packaging>`
增加如下依赖：

```xml
<dependency>

	<groupId>org.apache.maven</groupId>

	<artifactId>maven-plugin-api</artifactId>

	<version>3.5.0</version>

</dependency>

<dependency>

	<groupId>org.apache.maven.plugin-tools</groupId>

	<artifactId>maven-plugin-annotations</artifactId>

	<version>3.3</version>

</dependency>
```

新建一个类，继承 `org.apache.maven.plugin.AbstractMojo`，实现对应的方法
增加注解 `org.apache.maven.plugins.annotations.Mojo`，增加 name 属性，代表 plugin 的 goal
`mvn install`
参数传递：插件类中增加变量，增加注解 `org.apache.maven.plugins.annotations.Parameter`

### 使用插件
挂载在项目的 pom.xml 中，增加 `plugin` 中 `execution` 的 `phase` 和 `goal` 属性：

![markdown/20180724235147.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180724235147.png)



## 五、Maven Profile 动态配置文件
a) 使用场景 dev/test/pro

b) 根据 `activeProfile` 来切换 setting.xml 中设置的私服地址（家和公司两套）



## 六、Maven 仓库
a)下载

b)安装 解压

c)使用http://books.sonatype.com/nexus-book/reference3/index.html 

i.http://192.168.1.6:8081/nexus

ii.admin/admin123

d)发布

i.pom.xml 配置 

![markdown/20180724235259.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180724235259.png)

e)下载jar配置

i.配置mirror

ii.Profile



## 七、Maven Archetype 模板化
### 生成一个模板
- 在项目目录下执行命令：`mvn archetype:create-from-project`
- 命令运行成功后，会在工程的 `target/generated-sources/archetype` 目录下生成一个 Archetype，进入这个目录：`cd /target/generated-sources/archetype`
- 如果想要将新生成的 archetype 运行在本地仓库，就运行 maven 命令：`mvn install`；如果想要共享这个 archetype，就使用 deploy 命令。





## 八、Maven 反应堆
### 反应堆
在一个多模块的 Maven 项目中，反应堆（Reactor）是指所有模块组成的一个构建结果，对于单模块的项目，反应堆就是该模块本身，但是对于多模块的项目来说，反应堆就包含了各模块之间继承与依赖的关系，从而能够自动计算出合理的模块构建顺序。
构建顺序一般为：主 POM 的读取顺序 + 继承或者依赖的顺序
模块之间的依赖关系会将反应堆构成一个有向循环图（Directed Acyclic Graph，DAG），各个模块是该图的节点，依赖关系构成了有向边。这个图不允许出现循环，当出现循环依赖时，Maven 就会报错。
### 裁剪反应堆
有时，在多模块项目中，如果只改了某一个模块的内容，为了加快构建，可以不需要完整构建所有模块，可以有选择地构建，常用命令如下：
- -am，—also-make，表示同时构建所列模块的依赖模块
- -amd，-also-make-dependents，表示同时构建依赖于所列模块的模块
- pl，—projects <args>，表示构建指定的模块，模块间用逗号分隔
- rf，-resume-from <args>，表示从指定的模块开始构建

`mvn clean package  -Dmaven.test.skip=true -pl api -amd`