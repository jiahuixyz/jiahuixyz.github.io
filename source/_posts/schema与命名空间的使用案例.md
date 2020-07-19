---
title: schema与命名空间的使用案例
date: 2020-01-12 15:03:30
tags:
---

#### 1、xml如何使用schema

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
</beans>
```

解释：<br>
- 一份xml中，只能有一个省略别名的命名空间，如：<br>
**xmlns="http://www.springframework.org/schema/beans"** ；

- 省略别名和未省略别名的命名空间都需要在**xsi:schemaLocation**处填写实际的xsd地址；<br>
Spring这份xml引用xsd文件路径是网址，如果我们的不是网址的话可以这么写<br>
**xsi:schemaLocation="http://spec.nst.cn/specification/namespace schema.xsd "**（相对路径）；

- 另外**xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"** 是固定的，作用就是让我可以使用**xsi:schemaLocation** 这个属性；

#### 2、如何定义schema

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
	xmlns:nst="http://spec.nst.cn/specification/namespace"
	targetNamespace="http://spec.nst.cn/specification/namespace" elementFormDefault="qualified">
</xs:schema>
```

解释:<br>
- **targetNamespace="http://spec.nst.cn/specification/namespace"** 定义了一个命名空间，只有在schema中定义了命名空间，xml引用时才能使用别名，否则只能省略别名；

- **xmlns:nst="http://spec.nst.cn/specification/namespace"** 引用了一个命名空间，并起了一个别名，此处schema的身份相当于一个xml

- 一般schema中的通用写法为**elementFormDefault="qualified" attributeFormDefault="unqualified"** ，后续做解释

<br>
如有疑问，欢迎讨论~~