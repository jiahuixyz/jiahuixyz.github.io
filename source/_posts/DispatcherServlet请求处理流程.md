---
title: DispatcherServlet请求处理流程
date: 2020-03-07 17:29:14
tags:
---

1. 用户向服务器发送请求，请求被SpringMVC的前端控制器DispatcherServlet截获；
2. DispatcherServlet根据请求的URL,调用HandlerMapping获得对应的Handler以及Handler对应的Interceptor（拦截器），这些会被封装到一个HandlerExecutionChain中返回。
3. DispatcherServlet根据获得的Handler，选择一个合适的HandlerAdapter，HandlerAdapter使用Handler处理请求，调用实际处理的方法。
4. 提取请求中的模型数据，开始执行Handler。填充Handler参数时，根据配置，会进行下列工作：

- 数据格式化。如将字符串格式化为数字或日期。
- 数据验证。验证数据的格式、长度，验证结果存储到BindingResult或Error中。

5. Handler执行完成后，向DispatcherServlet返回一个ModelAndView，ModelAndView中包含视图和模型。
6. 根据ModelAndView，选择一个合适的ViewResolver（视图解析器）。
7. ViewResolver结合Model和View来渲染视图，将渲染结果返回给浏览器。

图1：DispatcherServlet继承结构

![The picture was not found.](https://jiahuixyz.github.io/img-service/img/DispatcherServlet-Diagram.png)

图2：DispatcherServlet初始化九大组件

![The picture was not found.](https://jiahuixyz.github.io/img-service/img/DispatcherServlet-Init.png)