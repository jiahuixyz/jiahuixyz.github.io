---
title: Java实现xml文件的xsd校验（schema校验）
date: 2020-01-12 15:03:30
tags:
---

>JDK中的javax.xml包中有能进行schema校验的类库，但只能返回true或false，无法给出确切的错误信息。

Dom4j中给出了几种schema校验的思路，本文实现其中一种。<br>
Dom4j在github上的文档地址是：[https://github.com/dom4j/dom4j/wiki/Cookbook](https://github.com/dom4j/dom4j/wiki/Cookbook/)

校验时，能够记录schema中所有不匹配的错误，但首先要保证xmL格式正确，否则只输出格式错误信息。

pom文件：
```xml
  <dependencies>
  	<!-- dom4j XML工具包 -->
		<dependency>
			<groupId>dom4j</groupId>
			<artifactId>dom4j</artifactId>
			<version>1.6.1</version>
		</dependency>
		<dependency>
			<groupId>jaxen</groupId>
			<artifactId>jaxen</artifactId>
			<version>1.1.4</version>
		</dependency>
  </dependencies>
```
<!--more-->
主要代码：
```java
    public static void validateXMLByXSD() {
        // 从类路径下读取文件
        URL xmlFileName = SchemaValidateUtil.class.getClassLoader().getResource("test.xml");
        // 输出为file:/D:/MyWorkSpace/FirstWorkSpace/validate-xml-test/target/classes/test_schema.xsd
        String xsdFileName = SchemaValidateUtil.class.getClassLoader().getResource("test_schema.xsd").toString();
        
        try {
            //创建默认的XML错误处理器
            XMLErrorHandler errorHandler = new XMLErrorHandler();
            //获取基于 SAX 的解析器的实例
            SAXParserFactory factory = SAXParserFactory.newInstance();
            //解析器在解析时验证 XML 内容。
            factory.setValidating(true);
            //指定由此代码生成的解析器将提供对 XML 名称空间的支持。
            factory.setNamespaceAware(true);
            //使用当前配置的工厂参数创建 SAXParser 的一个新实例。
            SAXParser parser = factory.newSAXParser();
            //创建一个读取工具
            SAXReader xmlReader = new SAXReader();
            //获取要校验xml文档实例
            Document xmlDocument = (Document) xmlReader.read(xmlFileName);
            //设置 XMLReader 的基础实现中的特定属性。核心功能和属性列表可以在 [url]http://sax.sourceforge.net/?selected=get-set[/url] 中找到。
            parser.setProperty("http://java.sun.com/xml/jaxp/properties/schemaLanguage",
                    "http://www.w3.org/2001/XMLSchema");
            parser.setProperty("http://java.sun.com/xml/jaxp/properties/schemaSource",xsdFileName);
            //创建一个SAXValidator校验工具，并设置校验工具的属性
            SAXValidator validator = new SAXValidator(parser.getXMLReader());
            //设置校验工具的错误处理器，当发生错误时，可以从处理器对象中得到错误信息。
            validator.setErrorHandler(errorHandler);
            //校验
            validator.validate(xmlDocument);

            // 错误输出方式1
            XMLWriter writer = new XMLWriter(OutputFormat.createPrettyPrint());
            //如果错误信息不为空，说明校验失败，打印错误信息
            if (errorHandler.getErrors().hasContent()) {
                System.out.println("XML文件通过XSD文件校验失败！");
                writer.write(errorHandler.getErrors());
            } else {
                System.out.println("XML文件通过XSD文件校验成功！");
            }
            
            // 错误输出方式2
            System.out.println();
            StringBuilder errorMsg = new StringBuilder();
			if (errorHandler.getErrors().hasContent()) {
				List<Element> elements = errorHandler.getErrors().elements();
				for (Element element : elements) {
					String line = String.valueOf(Integer.parseInt(element.attributeValue("line")) - 1);
					String text = element.getText();
					errorMsg.append("(第 " + line + "行出现错误) " + text+"\r\n");
				}
				errorMsg.append("XML文件通过XSD文件校验失败！");
				System.out.println(errorMsg.toString());
			}else {
				System.out.println("XML文件通过XSD文件校验成功！");
			}
        } catch (Exception ex) {
            System.out.println("XML文件: " + xmlFileName + " 通过XSD文件:" + xsdFileName + "检验失败。/n原因： " + ex.getMessage());
            ex.printStackTrace();
        }
    }
```