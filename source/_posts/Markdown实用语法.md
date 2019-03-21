---
title: Markdown实用语法
date: 2019-03-21 20:30:51
tags:
---


### 1、表格

示例：
```
item|value|qty
:-|:-:|-:|
computer|1600usd|5
phone|12usd|12
```

item|value|qty
:-|:-:|-:|
computer|1600usd|5
phone|12usd|12

### 2、分割线
示例：
`***或者---`

---

### 3、超链接

示例：`[百度网址](http://www.baidu.com)`

[百度网址](http://www.baidu.com)

### 4、列表

示例：
```
- 列表1
    1. 列表1
    2. 列表2
    3. 列表3
    4. 列表嵌套需要在子列表前加一个tab
+ 列表2
* 列表3
4. 列表4
```

- 列表1
    1. 列表1
    2. 列表2
    3. 列表3
    4. 列表嵌套需要在子列表前加一个tab
+ 列表2
* 列表3
4. 列表4

### 5、引用

示例：

`> 过去再也回不去。--马克多夫`

`>> 今天的午餐才是值得关心的。--克里斯多夫`

> 过去再也回不去。--马克多夫

>> 今天的午餐才是值得关心的。--克里斯多夫

### 6、字体加粗和倾斜

示例：
```
**加粗**

*斜体*
```


**加粗**

*斜体*

### 7、代码块

单行：\`public static void main()\`

`public static void main()`

多行：

\`\`\`java  
public static void mian(){  
    System.out.println("Hello World!");  
}  
\`\`\`

```java
public static void mian(){
    System.out.println("Hello World!");
}
```

>注意：单行和多行使用的都是**反引号`**，而不是**单引号'**