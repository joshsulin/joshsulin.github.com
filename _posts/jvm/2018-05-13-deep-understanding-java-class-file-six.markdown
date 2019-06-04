---
layout: post
title:  "深入理解Java Class文件格式(六)"
date:   2018-05-13 20:19:20 +0800
categories: Jvm
tags: Jvm
---

经过前几篇文章， 终于将常量池介绍完了， 之所以花这么大的功夫介绍常量池， 是因为对于理解class文件格式，常量池是必须要了解的， 因为class文件中其他地方，大量引用了常量池中的数据项。  对于还不了解常量池的读者， 如果想要深入了解class文件格式， 或者想继续读这篇博客和本专栏以后的博客， 那么我建议先把我前面的几篇博客读一下，把常量池的结构熟悉一下， 对于理解后面的内容很有帮助。 

虽然介绍完了常量池， 但是class文件中位于常量池下面的内容还有很多呢。 接下来， 我们就分析class文件中位于常量池下面的内容， 不用担心， 只要把常量池搞明白了， 这些内容就会很容易理解。

在开始进入正文之前， 在这里再次给出class文件的整体格式。 这个表格曾出现在 深入理解Java Class文件格式（一） 这篇文章中。 之所以把这个表格列在这里， 是想再次给读者一个class文件的总体概览。表格的内容如下：

| 类型     | 名称                | 数量  |
| -------- | :-----              | :---- |
| u4       | magic               | 1     |
| u2       | minor_version       | 1     |
| u2       | major_version       | 1     |
| u2       | constant_pool_count | 1     |
| cp_info  | constant_pool       | constant_pool_count -1 |
| u2       | access_flags        | 1     |
| u2       | this_class          | 1     |
| u2       | super_class         | 1     |
| u2       | interfaces_count    | 1     |
| u2       | interfaces          | interfaces_count |
| u2       | fields_count        | 1     |
| field_info | fields            | fields_count |
| u2       | methods_count       | 1     |
| method_info | methods          | methods_count |
| u2       | attribute_count     | 1     |
| attribute_info | attributes    | attributes_count |

下面我们就开始介绍class文件中的其他内容。

### class文件中的访问标志信息

从上面的表格可以看出， 位于常量池下面的2个字节是access_flags 。 access_flags 描述的是当前类（或者接口）的访问修饰符， 如public， private等， 此外， 这里面还存在一个标志位， 标志当前的额这个class描述的是类， 还是接口。access_flags 的信息比较简单， 下面列出access_flags 中的各个标志位的信息。本来写这个系列博客参考的是《深入java虚拟机》， 但是这本书比较老了， 关于java 5以后的新特性没有进行解释，这本书中指列出了5个标志值， 而最新的JVM规范是针对java 7 的， 其中加入了额外的三个标志位。 分别是ACC_SYNTHETIC， ACC_ANNOTATION 和 ACC_ENUM  。

| 标志名     | 标志值  | 标志含义   | 针对的对象 |
| --------   | :-----  | :----      | :----      |
| ACC_PUBLIC | 0x0001  | public类型 | 所有类型   |
| ACC_FINAL  | 0x0010  | final类型  | 类         |
| ACC_SUPER  | 0x0020  | 使用新的invokespecial | 类和接口 |
| ACC_INTERFACE | 0x0200  | 接口类型 | 接口 |
| ACC_ABSTRACT  | 0x0400  | 抽象类型 | 类和接口 |
| ACC_SYNTHETIC | 0x1000  | 该类不由用户代码生成 | 所有类型 |
| ACC_ANNOTATION | 0x2000  | 注解类型 | 注解 |
| ACC_ENUM | 0x4000  | 枚举类型 | 枚举 |

其他标志就不做介绍了， 这些标志都很简单。 读者感觉比较陌生的可能是ACC_SUPER这个标志。 读者会想， 类型不能被super关键字修饰啊， 那这个ACC_SUPER是做什么的呢？表中可以看出，　它的含义是：使用新的invokespecial语义 。 invokespecial是一个字节码指令， 用于调用一个方法， 一般情况下， 调用构造方法或者使用super关键字显示调用父类的方法时， 会使用这条字节码指令。 这正是ACC_SUPER这个名字的由来。 在java 1.2之前， invokespecial对方法的调用都是静态绑定的， 而ACC_SUPER这个标志位在java 1.2的时候加入到class文件中， 它为invokespecial这条指令增加了动态绑定的功能。 这里可能有几个概念读者不是很明白， 如静态绑定， 动态绑定等， 这些概念会在以后的博客中详细介绍。

还有一点需要说明， 既然access_flags 出现在class文件中的类的层面上， 那么它只能描述类型的修饰符， 而不能描述字段或方法的修饰符， 希望读者不要将这里的access_flags 和后面要介绍的方法表和字段表中的访问修饰符相混淆。

此外， 在Java 5 的中， 引入和注解和枚举的新特性， 那么可以推测，  ACC_ANNOTATION 和 ACC_ENUM是在Java 5版本中加入的。 class文件虽然总体上保持前后一致性， 但他也不是一成不变的， 也会跟着Java版本的提升而有所改变， 但是总体来说， class文件格式还是相对稳定的， 变动的地方不是很多。

### class文件中的this_class

访问标志access_flags 下面的两个字节叫做this_class， 它是对当前类的描述。 它的两个字节的数据是对常量池中的一个CONSTANT_Class_info数据项的一个索引。 CONSTANT_Class_info在上面的文章中已经介绍过了。 CONSTANT_Class_info中有一个字段叫做name_index ， 指向一个CONSTANT_Utf8_info ， 在这个CONSTANT_Utf8_info 中存放着当前类的全限定名。 

如果当前类为Person：

```java
package com.jg.zhang;
 
public class Person {
 
	int age;
 
	int getAge(){
		return age;
	}
}
```

将Person.class反编译后， 可以在常量池中看到如下两项：

```java

  Constant pool:
   #1 = Class              #2             //  com/jg/zhang/Person
   #2 = Utf8               com/jg/zhang/Person

```

这两项就是当前类的信息。 其中索引为1的CONSTANT_Class_info会被class文件中的this_class所引用。 下面给出示例图（其中虚线范围内表示常量池的区域）：

<a href="/img/post/jvm/20.png" target="_blank"><img src="/img/post/jvm/20.png" /></a>

### class文件中的super_class

super_class紧跟在this_class之后。 它和this_class一样是一个指向常量池数据项的索引。 它指向一个CONSTANT_Class_info， 这个CONSTANT_Class_info数据项描述的是当前类的超类的信息。CONSTANT_Class_info中的name_index指向常量池中的一个CONSTANT_Utf8_info ，CONSTANT_Utf8_info 中存放的是当前类的超类的全限定名。 如果没有显式的继承一个，也就是说如果当前类是直接继承Object的， 那么super_class值为0 。 我们在前面的文章中提到过， 如果一个索引值为0， 那么就说明这个索引不引用任何常量池中的数据项， 因为常量池中的数据项是从1开始的。 也就是说， 如果一个类的class文件中的super_class为0 ， 那么就代表该类直接继承Object类。

下面以代码来说明：

```java
package com.jg.zhang;
 
public class Programer extends Person{
 
	Computer computer;
	
	public Programer(Computer computer){
		this.computer = computer;
	}
	
	public void doWork(){
		computer.calculate();
	}
}
```

上面的Programer类继承自Person类。 那么反编译Programer .class ， 它的常量池中会存在如下信息：

```java
Constant pool:
 
.........
.........
 
   #3 = Class              #4             //  com/jg/zhang/Person
   #4 = Utf8               com/jg/zhang/Person
```

这两项就是当前类的父类的信息。 其中索引为3的CONSTANT_Class_info会被class文件中的super_class引用。 下面给出示例图（其中虚线范围内表示常量池的区域）：

<a href="/img/post/jvm/21.png" target="_blank"><img src="/img/post/jvm/21.png" /></a>

### class文件中的interfaces_count和interfaces

紧接着super_class的是interfaces_count， 表示当前类所实现的接口的数量或者当前接口所继承的超接口的数量。 注意， 只有当前类直接实现的接口才会被统计， 如果当前类继承了另一个类， 而另一个类又实现了一个接口， 那么这个接口不会统计在当前类的interfaces_count中。 在interfaces_count后面是interfaces， 他可以看做是一个数组， 其中的每个数组项是一个索引， 指向常量池中的一个CONSTANT_Class_info， 这个CONSTANT_Class_info又会引用常量池中的一个CONSTANT_Utf8_info ， 这个CONSTANT_Utf8_info 中存放着有当前类型直接实现或继承的接口的全限定名。 当前类型实现或继承了几个接口， 在interfaces数组中就会有几个数项与之相对应。

下面看代码示例：

```java
package com.jg.zhang;
 
public class Plane implements IFlyable, Cloneable{
 
	@Override
	public void fly() {
		
	}
}
```

Plane类实现了一个自定义的IFlyable接口， 还实现了一个JDK中的Cloneable接口， 那么它的常量池中会有如下信息：

```java

Constant pool:
 
.........
.........
 
   #5 = Class              #6             //  com/jg/zhang/IFlyable
   #6 = Utf8               com/jg/zhang/IFlyable
   #7 = Class              #8             //  java/lang/Cloneable
   #8 = Utf8               java/lang/Cloneable
   
.........
.........
```

这四项数据就是当前的Plane类所实现的接口的信息。 第五项和第六项描述了Plane所实现的IFlyable接口， 第七项和第八项描述了Plane所实现的接口Cloneable接口。 下面是示意图（其中虚线范围内表示常量池的区域）：

<a href="/img/post/jvm/22.png" target="_blank"><img src="/img/post/jvm/22.png" /></a>

### 总结

在本篇博客中， 继续讲解了class文件中常量池以下的部分。 主要讲解了三个部分， 分别是this_class ， super_class ， interfaces_count和interfaces 。 这三个数据项分别描述了当前类（就是当前class文件所在的类）， 当前类所继承的超类， 和当前类所实现的接口（如果当前class文件代表的是一个接口， 那么 interfaces_count和interfaces描述的是当前接口所继承的超接口）。

这几个数据项都持有指向常量池的索引。 真实的信息都是存放在常量池中的， 只不过常量池中的这些信息会被this_class ， super_class ， interfaces_count和interfaces 引用。 

通过本篇博客我们可以知道源文件中的当前类， 当前类的超类以及当前类的超接口在class文件中是如何被描述的。 在下一篇博客中， 将会讲解源文件中的定义的字段， 声明的方法在class文件中是如何描述的。 

原文: https://blog.csdn.net/zhangjg_blog/article/details/21888943
