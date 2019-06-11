---
layout: post
title:  "深入理解Java Class文件格式(八)"
date:   2018-05-17 20:19:20 +0800
categories: Jvm
tags: Jvm
---

在本专栏的第一篇文章 深入理解Java虚拟机到底是什么 中， 我们主要讲解了什么是虚拟机， 这篇博客是对JVM的一个概述。 在随后的几篇文章中，一直在讲解class文件格式。 在今天这篇博客中， 将会继续讲解class文件中的其他信息。 在本文中， 将会讲解class文件中的最后一部分， 属性（attributes） 。 这里的属性和源文件中的属性不是一个概念。 在源文件中， 我们把在类中定义的字段也叫做属性。 而class文件中的属性， 可以看做是存储一些额外信息的数据结构。 下面我们就来介绍属性。

### class文件中的attributes_count和attributes

attributes_count位于class文件中methods的下面。 它占两个字节， 存储的是一个整数值， 表示class文件中属性的个数。 

attributes_count下面的是attributes， 可以把它看做一个数组， 每个数组项是一个attribute_info ， 每个attribute_info表示一个属性。attributes中有attributes_count个attribute_info 。

需要说明的是， 属性会出现在多个地方， 不仅仅出现在顶层的ClassFile中， 也会出现在class文件中的数据项中， 如出现在field_info中， 用来描述特定字段的一些信息， 还可以出现在method_info中， 用来描述特定方法的一些信息。 （关于field_info和method_info已经在上面一篇博客中介绍过， 不明白的可以参考上面的博客： 深入理解Java Class文件格式（七））

属性（attribute_info）的大概格式是这样的：

<a href="/img/post/jvm/27.png" target="_blank"><img src="/img/post/jvm/27.png" /></a>

其中attribute_name_index占两个字节， 它是一个指向常量池数据项的索引。 它指向一个CONSTANT_Utf8_info ， 这个CONSTANT_Utf8_info 中存放的是当前属性的名字。

attribute_name_index下面的四个字节叫做attribute_length， 它表示当前属性的长度， 这个长度不包括前6个字节， 也就是说只包括属性真实信息（也就是info）的长度。

attribute_length下面的数据是info， 它的长度由上面提到的attribute_length指定， 它存放的是真实的属性数据。

下面我们会依次介绍一些重要属性， 相对不是很重要的属性会一笔带过。

### ClassFile中的SourceFile属性

首先介绍一个比较简单的属性：SourceFile。 该属性出现在顶层的class文件中。 它描述了该类是从哪个源文件中编译来的， 注意， 描述的是源文件， 而不是类， 一个源文件中可以存在多个类。 它的格式如下：

<a href="/img/post/jvm/28.png" target="_blank"><img src="/img/post/jvm/28.png" /></a>

前面说过， attribute_name_index指向常量池中的一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info 中存放的是这个属性的名字字符串， 即“SourceFile” 。 

attribute_length是属性信息的长度， 这里是2， 因为这个属性的info就两个字节。

sourcefile_index占两个字节， 这也是为什么attribute_length是2的原因。 sourcefile_index指向常量池中的一个CONSTANT_Utf8_info ， 这个CONSTANT_Utf8_info 中存放的是生成该类的源文件的文件名， 这里的文件名不包括路径部分。

下面举例说明， 示例代码：

```java
package com.jg.zhang;
 
public class Person {
 
	int age;
 
	int getAge(){
		return age;
	}
}
```

反编译后的相关信息：

```java
public class com.jg.zhang.Person
  SourceFile: "Person.java"
Constant pool:
.........
  #20 = Utf8               SourceFile
  #21 = Utf8               Person.java
 
.........
```

反编译结果中的  SourceFile: "Person.java"  一行是SourceFile属性的简单表示形式。 可以把它看做一个可读的attribute_info 。 下面常量池中的第20项的CONSTANT_Utf8_info是对这个属性的属性名（attribute_name_index）的描述 ， 第21项的CONSTANT_Utf8_info是对源文件的文件名的描述。

下面是图例， 注意， 虚线范围内表示常量池区域：

<a href="/img/post/jvm/29.png" target="_blank"><img src="/img/post/jvm/29.png" /></a> 

### ClassFile中的InnerClasses属性

InnerClasses是一个存在于顶层class文件中的属性， 它描述的是内部类和外围类的关系。  这是一个相对来说比较复杂的属性， 因为每个类可能有多个内部类， 而这些内部类中可能还有内部类， 多层嵌套。外围类中的InnerClasses属性必须描述它的所有内部类， 而内部类中的InnerClasses也必须描述它的外围类。 

由于这个属性相对较为复杂， 而对于我们理解class文件又不具有很大的意义， 所以我们只是简单的介绍一下。 如果想深入理解这个属性， 请参考 《深入Java虚拟机》 第144到166页。 

下面是这个属性的结构：

<a href="/img/post/jvm/30.png" target="_blank"><img src="/img/post/jvm/30.png" /></a> 

attribute_name_index和attribute_length就不过多介绍了， 和上面介绍的是一样的。

number_of_classes描述的是内部类的个数。

classes可以看做是一个数组， 这个数组中的每一项是一个inner_class_info， 而每个inner_class_info是对一个内部类的描述。每个 inner_class_info的结构如下：

<a href="/img/post/jvm/31.png" target="_blank"><img src="/img/post/jvm/31.png" /></a> 

### Synthetic属性

Synthetic属性可以出现在filed_info中， method_info中和顶层的ClassFile中， 分别表示这个字段， 方法或类不是有用户代码生成的（即不存在与源文件中）， 而是由编译器自动添加的。 例如， 编译器会为内部类增加一个字段， 该字段是对外部类对象的引用； 如果一个不定义构造方法， 那么编译器会自动添加一个无参数的构造方法<init>， 如果定义了静态字段或静态代码块， 还会根据具体情况， 增加静态初始化方法<clinit> 。 此外， 有些机制， 如动态代理， 会在运行时自动生成字节码文件， 由于这些类不是由源文件中编译来的， 所以这些类的class文件中会有一个Synthetic属性。 

它的结构如下：

<a href="/img/post/jvm/32.png" target="_blank"><img src="/img/post/jvm/32.png" /></a> 

可以看到， 它没有真正的属性数据info， 它只是一个标志性的属性， 用来表示它所在的字段， 方法或类是由编译器自动添加的 。 

下面以实例代码来说明， 源码如下：

```java
package com.jg.zhang;
 
public class Person {
	
	static{
		System.out.println("static");
		
	}
	
	int age;
 
	int getAge(){
		return age;
	}
}
```

反编译后的相关信息如下：

```java
{
  int age;
    flags:
 
 
  static {};
 
    .........
 
  public com.jg.zhang.Person();
    
    .........
 
  int getAge();
 
    .........
}
```

由反编译结果可以看出， 编译器自动生成了静态初始化方法和构造方法。 可能是因为Synthetic属性是可选的（也就是说某个版本的编译器可以选择不加入Synthetic属性） ，所以在反编译后的结果中没有发现Synthetic属性。

### ConstantValue属性

ConstantValue属性出现在class文件中的field_info中， 也就是说它是一个和字段相关的属性。 每个field_info中最多只能出现一个ConstantValue属性。 此外， 要注意的是， 必须是静态字段才可以有ConstantValue属性。 这个静态字段可以是final的， 也可以不是final的。 

这个属性为静态变量提供了另一种初始化的方式。 静态变量初始化的方式有两种， 一种就是现在要讲得ConstantValue属性， 另一种就是静态初始化方法<clinit> 不同的编译器和虚拟机可以有不同的实现方式。 但是如果虚拟机决定使用ConstantValue属性为静态变量赋值， 那么为这个变量的赋值动作， 必须位于执行<clinit>方法之前。 

此外， 只有基本数据类型或String类型的静态变量才可以存在ConstantValue属性， 原因在下面会有说明。 

下面介绍它的结构：

<a href="/img/post/jvm/33.png" target="_blank"><img src="/img/post/jvm/33.png" /></a>

attribute_name_index和attribute_length就不过多介绍了， 和上面介绍的是一样的。这里的attribute_length为2 。 

位于attribute_length之下的是constantvalue_index ， 这是一个指向常量池中某个数据项的索引。这个常量池数据项中存放的就是当前字段的值。

这个常量池中的数据项，根据field_info描述的字段的不同， 可以是不同类型的数据项， 如果当前字段是byte， short， char， int， boolean类型， 那么这个被指向的常量池数据项就会是一个CONSTANT_Integer_info ， 如果当前字段是一个long类型的字段， 那么这个被指向的常量池数据项就会是一个CONSTANT_Long_info。 如果当前字段是是一个String类型的字段 ， 那么这个被指向的常量池数据项就是一个CONSTANT_String_info 。 这里有一点需要说明， 虽然java语言支持byte， short， char， boolean类型， 但是JVM却不支持这几种类型， 表现在class文件中就是， class文件中的常量池中没有和这几个数据类型相对应的数据项， 这几中类型都被JVM在执行时当做int来对待， 表现在class文件中就是， 这几种类型都对应常量池中的CONSTANT_Integer_info 数据项。 

这也说明了， 为什么只有基本数据类型和String类型的静态常量才会存在ConstantValue属性 。 因为constantvalue_index只是一个指向常量池的索引， 而其他引用类型的常量不会存在于常量池中。

下面以实例来说明， 实例代码如下：

```java
package com.jg.zhang;
 
public class Person {
	
	static final int a = 1;
	
	int age;
 
	int getAge(){
		return age;
	}
}
```

反编译后的相关结果如下：

```java
......
 
Constant pool:
 
   #7 = Utf8               ConstantValue
   #8 = Integer            1
 
 
{
  static final int a;
    flags: ACC_STATIC, ACC_FINAL
    ConstantValue: int 1
 
    .........
}
```

<a href="/img/post/jvm/34.png" target="_blank"><img src="/img/post/jvm/34.png" /></a>

可以看到， 源文件中的a字段， 是static final 的， 所以编译器为这个字段的filed_info生成了ConstantValue属性。 这个属性的示意图如下所示， 注意， 虚线范围内表示常量池区域：

### Deprecated属性

Deprecated属性可以存在于filed_info中， method_info中和顶层的ClassFile中， 分别表示这个字段， 方法或类已经过时。 这个属性用来支持源文件中的@deprecated注解。 也就是说， 如果在源文件中为一个字段， 方法或类标注了@deprecated注解， 那么编译器就会在class文件中为这个字段， 方法或类生成一个Deprecated属性 。

Deprecated属性的格式如下：

<a href="/img/post/jvm/35.png" target="_blank"><img src="/img/post/jvm/35.png" /></a>

和上面的属性一样， attribute_name_index属性指向一个常量池中的CONSTANT_Utf8_info 。 这个CONSTANT_Utf8_info中存放着该属性的名字 “Deprecated” 。 

attribute_length永远为0 ， 因为这个属性只是一个标志信息， 用来表示字段， 方法， 类已经过时， 而不具有任何实质性的属性信息。

下面以代码示例来说明， 代码如下：

```java
package com.jg.zhang;
 
public class Person {
	
	int age;
 
	@Deprecated
	int getAge(){
		return age;
	}
}
```

在getAge方法上使用了@deprecated 。 下面是反编译之后的相关信息：

```java
  ......
  
Constant pool:
  ......
 
  #18 = Utf8               Deprecated
 
  ......
 
{
 
  ......
 
  int getAge();
    flags:
    Deprecated: true
 
    ......
 
}
```

可以看到， 在getAge方法相关的信息中， 有一行 Deprecated: true ， 这说明编译器在getAge方法的method_info中加入了Deprecated属性。 常量池第18项的CONSTANT_Utf8_info中存放的是Deprecated属性的属性名“Deprecated” 。 

下面是示意图， 虚线范围内表示常量池区域：

<a href="/img/post/jvm/36.png" target="_blank"><img src="/img/post/jvm/36.png" /></a>

### 总结

本文就到此为止。 在本文中， 主要讲解了class文件中的一些属性。 这些属性可以出现在class文件中的对个地方， 用来描述一些其他信息。 

原文: https://blog.csdn.net/zhangjg_blog/article/details/22205831
