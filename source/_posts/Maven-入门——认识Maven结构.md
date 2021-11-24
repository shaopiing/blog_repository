---
title: Maven 入门——认识Maven结构
date: 2018-07-23 23:23:35
tags: "Program"
---

### 1、settings.xml 元素解读
**localRepository**
该元素表示本地 Maven 仓库的地址，不设置的话，默认为 `~/.m2/repository`
**pluginGroups**
将插件的信息注册到 Maven 中，是的执行 Maven plugin 命令的时候可以不指定 groupId 和 artifactId，比如：

![markdown/20180723234107.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180723234107.png)
这个生成源代码的插件，运行的时候不需要指定它的 groupId 和 artifactId，只需要执行`mybatis-generator:generate` 即可，因为这个插件的信息属于默认的两个插件组 `org.apache.maven.plugins`和`org.codehaus.mojo`其中的一个，如果是其他的话，则需要显式地配置一下。
**servers**
配置的私服的登录信息，比如 `username`、`password`等服务器的认证信息，也可以设置权限信息。
**mirrors**
远端的中央仓库，有时候下载第三方的 jar 包比较慢，可以更改为国内的一些镜像仓库，比如阿里云的仓库：

```xml
<mirror>

	<id>alimaven</id>

	<name>aliyun maven</name>

	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>

	<mirrorOf>central</mirrorOf>

</mirror>
```

如果是公司内部使用的 jar 包，可以放在自己搭建的私服上面，这里配置成自己的私服地址即可。
**profiles**、**activeProfiles**
其作用主要用于区分环境用的，也可以定义一些仓库，用来搜索需要的发布版或者快照版来构建，这个配置中的 profile，如果在激活列表里边存在，则这些 profile 将会覆盖 pom.xml 中的相同 id 的 profile。

### 2、pom.xml 元素解读
**modelVersion**
如果使用的是 Maven 3.0 及以上的版本，这里的值默认都是 4.0.0，而这个值来自哪里呢，在 Maven 安装目录里 lib 目录里边的 `/lib/maven-model-builder-3.5.4.jar`，解压之后，进入到目录 `org/apache/maven/model` 里边，会看到一个 pom.xml 文件：

![markdown/20180723234751.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180723234751.png)
而且，继续往下看，你就会发现，为什么用 IDEA 生成的 Maven 项目，它的目录结构是约定好的，约定的配置就是这个 super pom，这样只要是使用 Maven 开发的项目，其目录结果都是一样的，这种思想就是常常听说的**Convention Over Configuration**（约定优于配置）。
**groupId**
定义当前 Maven 项目隶属的实际项目，由于经常会有多模块的 Maven 项目，所以 Maven 项目与实际项目不一定是一对一的关系，因此，`groupId` 不应该对应项目隶属的组织或公司，应该到具体的项目；实际的表示方式也应该与 Java 包名的表示方式类似，通常与域名反向一一对应。
**artifactId**
该元素定义实际项目中的一个 Maven 项目（模块），推荐的做法是使用实际项目名称作为 artifactId 的前缀，这样做的好处是方便寻找实际构件。
**version**
该元素定义 Maven 项目当前所处的版本。
**packaging**
打包方式，默认为 jar。
**properties**
用于定义一些配置常量，比如依赖的版本号。
**dependencyManagement**
该元素只能出现在父 pom.xml 中，其作用是为了统一版本号，而且这里的依赖只是一个声明，子 pom.xml 里用到其中某一个的时候，再去显式地引用。
**dependency**
引用依赖，其中的配置有如下几个：

- type，默认为 jar
- scope 表示其作用范围，有如下几个范围：
  - *compile*，编译时依赖，也是默认的依赖范围，编译、测试和运行都需要，比如** spring-core**
  - *test*，测试时依赖，只在测试阶段需要，比如** spring-test**
  - *provided*，编译时依赖，只在编译时需要，比如**servlet**
  - *runtime*，运行时依赖，只在运行时需要，比如**JDBC驱动类**
  - *system*，本地的一些 jar，比如短信的 jar 包，常用 ** systemPath**一起使用
- exclusions，用来** 排除**由于传递依赖引入（参考第三小节）的但是是不需要的依赖
- optional，可选依赖，默认为 false，用于放置依赖传递，当一个项目 A 依赖另一个项目 B 时，项目 A 可能很少一部分功能用到了项目 B，此时就可以在 A 中配置对 B 的可选依赖y
### 3、传递性依赖
传递性依赖可以减少一些引用的依赖，可以进行隐式地依赖，但是如果需要控制版本，最好的方式是，先排除该依赖，再显式地引用该依赖，依赖关系如下图：
![markdown/20180723234822.png](http://p8bc1hri5.bkt.clouddn.com/markdown/20180723234822.png)
### 4、依赖仲裁
如果根据传递性依赖，同时依赖了两次某一个 jar 包，例如，项目 A 有这样的依赖关系：A -> B -> C -> X(1.0)、A -> D -> X(2.0)，X 是 A 的传递性依赖，那么哪个 X 会被 Maven 解析使用呢，Maven 依赖仲裁（Maven Mediation）的第一原则是：**路径最近者优先**，该例中 X(1.0) 的路径长度为 3，而 X(2.0) 的路径长度为 2，因此 X(2.0) 会被解析使用。
再比如这样的依赖关系：A -> B -> Y(1.0)，A -> C -> Y(2.0)，Y(1.0) 和 Y(2.0) 的依赖路径长度是一样的，都为 2，根据 Maven 依赖仲裁（Maven Mediation）的第二原则：**第一声明者优先**，在 POM 中依赖声明的顺序决定了谁会被解析使用，顺序最靠前的那个依赖优先被解析，所以 Y(2.0) 就会被解析使用。
### 5、优化依赖
`mvn dependency:analyze`
使用但未声明的依赖（Used undeclared dependencies），建议显示声明
声明但未使用的依赖（Unused declared dependencies），有可能是运行时使用的