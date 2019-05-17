---
layout: post
title:  "maven-assembly-plugin插件使用说明"
date:   2019-05-17 15:35:15 +0800
categories: Maven
tag: Maven
---

### 引言

对于Spring-Boot项目而言, maven-jar-plugin、spring-boot-maven-plugin 这两个插件是必用的, 都作用于 package 阶段. 最终结果都是jar包. 这篇博客将引入一个作用于package阶段的另一个插件 maven-assembly-plugin.

### 插件用途

maven-assembly-plugin 插件主要是聚合项目的输出，比如依赖，模块以及其他文件。通俗的来说，就是将项目内容按照一定规则及指定格式重新组合并输出. 比如说 公司要求线上自动化部署要求包为 tar.gz 格式的, 以便于将启动命令放在一个shell脚本里面.

### 插件使用

```java
<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-assembly-plugin</artifactId>
		<version>3.1.0</version>
		<executions>
				<execution>
						<id>assembly</id>
						<phase>package</phase>
						<goals>
								<goal>single</goal>
						</goals>
				</execution>
		</executions>
</plugin>
```

上述配置中，除了基本的插件坐标声明外，还有插件执行配置，executions下每个execution子元素可以用来配置执行一个任务. 该例中配置了一个id为assembly的任务，通过phrase配置，将其绑定到package生命周期阶段上，再通过goals配置指定要执行的插件目标. 

其实针对大多数插件而言, 按照上述配置就可以了, 但是该插件的用途是聚合一些资源进行归档, 还涉及到指定哪些资源并归档到哪个目录中, 所以还涉及到一个配置文件. 称之为描述符

```java
<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-assembly-plugin</artifactId>
		<version>3.1.0</version>
		<executions>
				<execution>
						<id>assembly</id>
						<phase>package</phase>
						<goals>
								<goal>single</goal>
						</goals>
				</execution>
		</executions>
		<configuration>
				<appendAssemblyId>false</appendAssemblyId>
				<descriptors>
						<descriptor>assembly/descriptor.xml</descriptor>
				</descriptors>
		</configuration>
</plugin>
```

在描述符文档中, 其实不难, 一般就是书写一些 <fileSet></fileSet>, 指定将哪个资源放在哪个目录下.

该插件详细说明和使用 见 <a href="https://maven.apache.org/plugins/maven-assembly-plugin/" target="_blank">官方文档</a>
