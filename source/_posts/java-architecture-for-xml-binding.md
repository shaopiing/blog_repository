---
title: "知识百科系列 (二) —— JAXB"
date: 2017-08-15 00:04:13
category: 程序人生
tags: [Program,WikiKnowledge]
---
> 最近因为工作需要，有一些把数据转化成 XML 文件的需求，所以就研究了下 JAXB。

JAXB（Java Architecture for XML Binding 简称 JAXB）允许 Java 开发人员将 Java 类映射为 XML 表示方式。
JAXB 提供两种主要特性：将一个 Java 对象序列化为 XML，以及反向操作，将 XML 解析成 Java 对象。
换句话说，JAXB 允许以 XML 格式存储和读取数据，而不需要程序的类结构实现特定的读取 XML 和保存 XML 的代码.

**1、编组（Marshal）**

JAXB 提供的基于注解形式的相互转化，配置起来很简单，下面是配置指出如何对应的关系：

```java
@XmlType(propOrder = {"city", "firstName", "lastName", "postalCode"})

@XmlRootElement(name = "Person")

public class Person {

    private String firstName;

    private String lastName;

    private Integer postalCode;

    private String city;

    public String getFirstName() {
        return firstName;
    }
    
    @XmlElement(name = "Person_FirstName")
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return lastName;
    }
    
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    
    public Integer getPostalCode() {
        return postalCode;
    }
    
    public void setPostalCode(Integer postalCode) {
        this.postalCode = postalCode;
    }
    
    public String getCity() {
        return city;
    }
    
    @XmlAttribute(name = "city", required = true)
    public void setCity(String city) {
        this.city = city;
    }

 }
```

说明一下几个注解的意思：
*   `@XmlRootElement` 表示根元素
*   `@XmlElement` 来与 setter 方法一起使用
*   `@XmlAttribute` 用来传递属性到 XML 节点，可以用一些 property 描述
*   `@XmlType` 来表示一些特殊选项，比如 propOrder 表示排序

接着我们就可以从 Java 对象生成 XML 文件：

![](http://p8bc1hri5.bkt.clouddn.com/java-architecture-for-xml-binding-1.png)

通过 main 方法调用生成 XML 文件：

```java
public static void main(String[] args) {

    try {

        Person person = new Person();

        person.setFirstName("com");
        person.setLastName("demo");
        person.setCity("Shanghai");
        person.setPostalCode(200000);
    
        marshallTest(person);
    } catch (JAXBException e) {
        e.printStackTrace();
    }

}
```

生成的 XML 如下图：

![](http://p8bc1hri5.bkt.clouddn.com/java-architecture-for-xml-binding-2.png)

最重要的类就是 `javax.xml.bind.JAXBContext` 的使用，更多关于此类的信息可以查看
[JAXB API](https://docs.oracle.com/javase/7/docs/api/javax/xml/bind/JAXBContext.html)

不仅能映射一个对象到 XML，还可以映射一组对象到 XML：

```java
@XmlRootElement(name = "Persons")

public class Persons {

    private ListPerson> persons;

    public List&lt;Person> getPersons() {
        return persons;
    }
    
    @XmlElement(name = "Person")
    public void setPersons(List&lt;Person> persons) {
        this.persons = persons;
    }
    
    @Override
    public String toString() {
        return "Persons{" +
                "persons=" + persons +
                '}';
    }

}
```

同理，main 方法调用也可以生成 XML：

```java
public static void main(String[] args) {

    try {

        Persons persons = new Persons();

        ListPerson> list = new ArrayList<>();

        Person person;

        for (int i = 0; i < 3; i++) {
            person = new Person();
            person.setFirstName("com" + i);
            person.setLastName("demo" + i);
            person.setCity("Shanghai" + i);
            person.setPostalCode(200000 + i);
            person.setBirthday(LocalDate.of(2017, 7, 16 + i));
            list.add(person);
        }
    
        persons.setPersons(list);
    
        marshallTest(persons);
    } catch (JAXBException e) {
        e.printStackTrace();
    }

}
```

生成的 XML 如下：

![](http://p8bc1hri5.bkt.clouddn.com/java-architecture-for-xml-binding-3.png)

**2、反编组（Un-marshal）**

反编组 XML 到 Java 对象和编组没太大区别，还是用的 `javax.xml.bind.JAXBContext` 类，只是用到的方法是 `createUnmarshaller()`：

![](http://p8bc1hri5.bkt.clouddn.com/java-architecture-for-xml-binding-4.png)

写个 main 方法生成一下：

```java
public static void main(String[] args) {

    try {

        File file = new File("person.xml");

        unMarshallTest(file, new Persons());

    } catch (JAXBException e) {

        e.printStackTrace();

    }

}
```

结果显而易见：

```java
Persons{persons=[Person{firstName='com0', lastName='demo0', postalCode=200000, city='Shanghai0', birthday=2017-07-16}, Person{firstName='com1', lastName='demo1', postalCode=200001, city='Shanghai1', birthday=2017-07-17}, Person{firstName='com2', lastName='demo2', postalCode=200002, city='Shanghai2', birthday=2017-07-18}]}
```

**3、适配器（Adapters）**

有些复杂类型（比如时间类型），JAXB 无法直接处理，我们需要编写一个适配器来告诉 JAXB 该如何处理。

```java
public class DateAdapters extends XmlAdapter<String, LocalDate> {

    @Override

    public LocalDate unmarshal(String date) throws Exception {

        return LocalDate.parse(date);

    }

    @Override
    public String marshal(LocalDate date) throws Exception {
        return date.toString();
    }

}
```

在上面的 Person 类中增加生日（birthday）属性，并增加注解 `@XmlJavaTypeAdapter` 来使用适配器：

![](http://p8bc1hri5.bkt.clouddn.com/java-architecture-for-xml-binding-5.png)

最终生成的 XML 则正确输出了生日：

![](http://p8bc1hri5.bkt.clouddn.com/java-architecture-for-xml-binding-6.png)

以上便是利用 JAXB 对 Java 与 XML 直接进行的转化，更多高级的内容，请查看
[用于 Java 和 XML 绑定的 JAXB 教程](https://www.javacodegeeks.com/2015/04/%E7%94%A8%E4%BA%8Ejava%E5%92%8Cxml%E7%BB%91%E5%AE%9A%E7%9A%84jaxb%E6%95%99%E7%A8%8B.html)