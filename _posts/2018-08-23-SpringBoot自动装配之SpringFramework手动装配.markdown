---
layout: post
title:  "SpringBoot自动装配之SpringFramework手动装配"
date:   2018-08-23 20:19:20 +0800
categories: SpringBoot
tags: ['SpringBoot', 'Java']
---
### 前言

最近在学习SpringBoot自动装配的实现原理, 通过一段时间的探索, 发现SpringBoot的自动装配是源于SpringFramework的手动装配。所以先对SpringFramework的手动装配做一个总结。

### 回顾SpringFramework手动装配如何使用

在我们刚开始接触Spring的时候, 要定义bean的话需要在xml中编写，比如:

```java
<bean id="myBean" class="your.pkg.YourClass"/>
```

后来发现如果bean比较多，需要写很多的bean标签，太麻烦了。于是出现了一个component-scan注解。这个注解直接指定包名就可以，它会去扫描这个包下所有的class，然后判断是否解析：

```java
<context:component-scan base-package="your.pkg"/>
```

再后来，由于注解Annotation的流行, 出现了@ComponentScan注解，作用跟component-scan标签一样:

```java
@ComponentScan(basePackages = {"your.pkg", "other.pkg"})
public class Application { ... }
```

功能其实很简单, context:component-scan 或者 @ComponentScan 告诉Spring哪个packages下的用注解(@Component、@Service、@Controller、@Repository等等)标识的类会被spring自动扫描并且装入IOC容器。
   
### 3、假如需要我们自己实现该功能需要如何实现? 功能关键点有哪些？每个功能关键点如何实现?

* a、根据basePackages, 找到包含包名的绝对目录。
* b、根据绝对目录, 罗列出该目录下所有的Class文件。
* c、解析该Class文件, 判断是否包含特定的注解(@Component、@Service、@Controller、@Repository等等)的类。
* d、将满足条件的类, 装入IOC容器。

#### 3.1 如何根据basePackages, 找到包含包名的绝对目录。
需要完成: basePackages的值为 -> com.josh.demo.service 找到 /Users/project/java/xxx_project/target/classes/com/josh/demo/service?

实例代码片段如下:
```java
String basePackage = "com.josh.toefl.service";
// 转化成 com/josh/toefl/service/
String packageSearchPath = StringUtils.replace(basePackage, ".", "/") + "/";
Enumeration<URL> resources = Thread.currentThread().getContextClassLoader().getResources(packageSearchPath);
while (resources.hasMoreElements()) {
    URL url = resources.nextElement();
    System.out.println(url.getPath());
}
```

#### 3.2 根据找到包含包名的绝对目录, 如何罗列出该目录下所有的Class文件.

实例代码片段如下:
```java
String path = "/Users/project/java/xxx_project/target/classes/com/josh/toefl/service";
File dir = new File(path);
File[] files = dir.listFiles();
for (File file : files) {
    System.out.println(file.getAbsoluteFile());
}
```

#### 3.3 解析该Class文件, 判断是否包含特定的注解(@Component、@Service、@Controller、@Repository等等)的类。

实例代码片段如下:
```java
String fileName = "/Users/project/java/xxx_project/target/classes/com/josh/toefl/service/TfUserService.class";
Resource resource = new FileSystemResource(fileName);
SimpleMetadataReaderFactory simpleMetadataReaderFactory = new SimpleMetadataReaderFactory();
MetadataReader metadataReader = simpleMetadataReaderFactory.getMetadataReader(resource);
AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
String annotationTypeName = "org.springframework.stereotype.Component";
// 判断是否直接包含Component注解或者包含Component元注解
if (annotationMetadata.hasAnnotation(annotationTypeName) || annotationMetadata.hasMetaAnnotation(annotationTypeName)) {
    System.out.println("满足装配进IOC容器条件");
} else {
    System.out.println("不满足装配进IOC容器条件");
}
```

#### 3.4 将满足条件的类, 装入IOC容器。

实例代码片段如下:
```java
String fileName = "/Users/project/java/xxx_project/target/classes/com/josh/toefl/service/TfUserService.class";
Resource resource = new FileSystemResource(fileName);
SimpleMetadataReaderFactory simpleMetadataReaderFactory = new SimpleMetadataReaderFactory();
MetadataReader metadataReader = simpleMetadataReaderFactory.getMetadataReader(resource);
ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
sbd.setResource(resource);
sbd.setSource(resource);
// sbd 是根据class的metadata构建的 BeanDefinition, 中间还需要补充一些BeanDefinition, 然后通过 registerBeanDefinition 注入到IOC容器
```

### 4、Spring 中该功能是如何实现的?
以下源码以 context:component-scan xml 元素为例, 看看Spring中是如何实现的。

4.1 主要方法入口(标签解析)
/org/springframework/spring-context/4.3.8.RELEASE/spring-context-4.3.8.RELEASE-sources.jar!/org/springframework/context/annotation/ComponentScanBeanDefinitionParser.java
```java
@Override
public BeanDefinition parse(Element element, ParserContext parserContext) {
    // 获取到 base-package 的包名
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
    // 对包名进行处理
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
    // 对多个包名进行分隔, 转化成数组
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
    ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

    // Actually scan for bean definitions and register them.
    ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
    // 扫描包对应目录下的所有文件, 将符合条件的类转化成BeanDefinition
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
    registerComponents(parserContext.getReaderContext(), beanDefinitions, element);
    return null;
}
```

大家可以在这个方法里打一个断点, 逐步跟进。相信大家可以一探究竟。
