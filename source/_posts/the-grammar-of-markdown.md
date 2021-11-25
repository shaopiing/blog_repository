---
title: "知识百科系列 (四) —— Markdown 的基本语法汇总"
date: 2017-11-30 00:49:13
category: 程序人生
tags: [Program,WikiKnowledge]
---
Markdown入门。
<!--more-->

### 1、标题

在文本前面增加 `#` 即可，同理，你还可以增加二级标题、三级标题、四级标题、五级标题和六级标题，总共六级，只需要增加 `#` 即可，标题字号相应降低。例如：

```properties
# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

###### 六级标题
```

如下图所示：

![](http://p8bc1hri5.bkt.clouddn.com/the-grammar-of-markdown-1.png)

注：`#` 和【标题】之间建议保留一个字符的空格，这个最标准的 Markdown 写法。

### 2、列表

列表格式也是很常用的，只需在文字前面加上 `-` 即可，例如：

```properties
- 文本1
- 文本2
- 文本3
```

如果希望有序列表，也可以在文字前面加上 `1.` `2.` `3.` 就可以了，例如：

```properties
1. 文本1
2. 文本2
3. 文本3
```

如下图所示：

![](http://p8bc1hri5.bkt.clouddn.com/the-grammar-of-markdown-2.png)

注：`-`、 `1.` 和文本之间要保留一个字符的空格。

### 3、链接和图片

在 Markdown 中，插入链接不需要其他按钮，只需要使用 `[显示文本](链接地址)` 这样的语法即可，例如：

```markdown
[日拱一卒，功不唐捐](http://www.zhangsubiin.com)
```

在 Markdown 中，插入图片不需要其他按钮，只需要使用 `![](图片链接地址)` 这样的语法即可，例如：

```markdown
![](http://www.zhangsubiin.com/image/image.png)
```

如下图所示：

![](http://p8bc1hri5.bkt.clouddn.com/the-grammar-of-markdown-3.png)

注：插入图片和语法和链接的语法很像，只是前面多了一个 `!`。

### 4、引用

有时候会引用他人的文字，这时引用的格式就很有必要了，在 Markdown 中，你只需要在你希望引入的文字前面加上 `>` 就好了，例如：

```markdown
> 世界不是苟且，世界是远方，行万里路，才能回到内心深处。
```

最终显示的如下：

> 世界不是苟且，世界是远方，行万里路，才能回到内心深处。

### 5、粗体和斜体

Markdown 的粗体和斜体也比较简单，用两个 `*` 包含一段文本就是粗体的语法，用一个 `*` 包含一段文本就是斜体的语法，例如：

```markdown
*世界不是苟且*，世界是远方，**行万里路**，才能回到内心深处。
```

最终显示的就是下文，其中 “世界不是苟且” 是斜体，“行万里路” 是粗体：

*世界不是苟且*，世界是远方，**行万里路**，才能回到内心深处。

### 6、代码引用

需要引入代码时，如果引用的语句只有一段，不分行，可以用 ` 将语句包起来。
如果引用的语句为多行，可以将三个 ` 号置于这段代码的首行和末行。
如下图所示：

![](http://p8bc1hri5.bkt.clouddn.com/the-grammar-of-markdown-4.png)

### 7、表格

相关代码：

```markdown
a | b | c
--|---|---
d | e | f 
g | h | i
j | k | l
```

显示效果如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-grammar-of-markdown-5.png)

以及左右对齐的写法：

```
| 列1              | 列2                | 列3              |
| -----------------|:-----------------:| ----------------:|
| 第一列默认为居左对齐 | 第一列默认为居左对齐  | 第一列默认为居左对齐|
| 第二列为居中对齐    | 第二列为居中对齐     | 第二列为居中对齐    |
| 第三列为居右对齐    | 第三列为居右对齐     | 第三列为居右对齐    |
```

显示效果如下：

![](http://p8bc1hri5.bkt.clouddn.com/the-grammar-of-markdown-6.png)


### 参考与引用
- <a href="http://www.jianshu.com/p/q81RER" target="_blank">献给写作者的 Markdown 新手指南</a>
- <a href="http://wowubuntu.com/markdown/" target="_blank">Markdown 语法说明</a>