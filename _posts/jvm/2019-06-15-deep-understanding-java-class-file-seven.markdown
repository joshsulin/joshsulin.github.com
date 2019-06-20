---
layout: post
title:  "深入理解Java Class文件格式(七)"
date:   2019-05-15 20:19:20 +0800
categories: Jvm
tags: Jvm
---

前面的一系列博客， 对Class文件中的一部分数据项进行了介绍。 本文将会继续介绍class文件中未讲解的信息。 先回顾一下上面一篇文章。 在上一篇博客中， 我们介绍了：

this_class    对当前类的描述

super_class    对当前类的超类的描述

interfaces_count    当前类直接实现的接口的数量或当前接口直接继承的接口的数量

interfaces  对当前类或当前接口直接实现或继承的所有接口的描述

下面继续介绍class文件中的其他信息。

### class文件中的fields_count和fields

fields_count描述的是当前的类中定义的字段的个数， 注意， 这里包括静态字段， 但不包括从父类继承的字段。 如果当前class文件是由一个接口生成的， 那么这里的fields_count描述的是接口中定义的字段， 我们知道， 接口中定义的字段默认都是静态的。此外要说明的是， 编译器可能会自动生成字段， 也就是说， class文件中的字段的数量可能多于源文件中定义的字段的数量。 举例来说， 编译器会为内部类增加一个字段， 这个字段是指向外围类的对象的引用。

位于fields_count下面的数据叫做fields， 可以把它看做一个数组， 数组中的每一项是一个field_info 。这个数组中一共有fields_count个field_info ， 每个field_info都是对一个字段的描述。 下面我们详细讲解field_info的结构。 每个field_info的结构如下：

<a href="/img/post/jvm/23.png" target="_blank"><img src="/img/post/jvm/23.png" /></a>

####（1）access_flags

其中access_flags占两个字节， 描述的是字段的访问标志信息。 这里就不在详细介绍了， 下面给出一张表格（该表格来自《深入Java虚拟机》）：

| 标志位名称    | 值      | 含义                | 设定者   |
| --------      | :-----  | :----               | :----    |
| ACC_PUBLIC    | 0x0001  | 字段被设为public    | 类和接口 |
| ACC_PRIVATE   | 0x0002  | 字段被设为private   | 类       |
| ACC_PROTECTED | 0x0004  | 字段被设为protected | 类       |
| ACC_STATIC    | 0x0008  | 字段被设为static    | 类和接口 |
| ACC_FINAL     | 0x0010  | 字段被设为final     | 类和接口 |
| ACC_VOLATILE  | 0x0040  | 字段被设为volatile  | 类 |
| ACC_TRANSIENT | 0x0080  | 字段被设为transient | 类 |

####（2）name_index

access_flags下面的两个字节是name_index， 这是一个指向常量池的索引， 它描述的是当前字段的字段名。 这个索引指向常量池中的一个CONSTANT_Utf8_info数据项。 这个CONSTANT_Utf8_info数据项中存放的字符串就是当前字段的字段名。 

####（3）descriptor_index

name_index下面的两个字节叫做descriptor_index ， 它同样是一个指向常量池的索引， 它描述的是当前字段的描述符。 这个索引指向常量池中的一个CONSTANT_Utf8_info数据项。 这个CONSTANT_Utf8_info数据项中存放的字符串就是当前字段的描述符（关于字段描述符， 在前面的博客中已经有过详细的讲解， 如果不明白， 请参考前面的博客：深入理解Java Class文件格式（二））。 

####（4）attributes_count和attributes

descriptor_index 下面是attributes_count和attributes 。 这是对当前字段所具有的属性的描述。 这里的属性和源文件中的属性不是同一个概念， 在源文件测层面中， 属性是字段的另一种叫法， 希望读者不要疑惑。读者也不要轻视class文件中的属性， 这些属性可以描述很多的信息。 我们会在后面的文章中进行介绍。 

attributes_count表示这个字段有几个属性。attributes 可以看成一个数组， 数组中的每一项都是一个attribute_info ， 每个attribute_info 表示一个属性， 数组中一共有attributes_count个属性。可以出现在filed_info中的属性有三种， 分别是ConstantValue， Deprecated， 和 Synthetic。 这些属性会在后面的文章中进行介绍。

下面我们以代码的形式进行解释， 源码如下：

```java
package com.jg.zhang;
 
public class Programer extends Person{
 
	
	private Computer computer;
	
	public Programer(Computer computer){
		this.computer = computer;
	}
	
	public void doWork(){
		computer.calculate();
	}
}
```

反编译之后， 常量池中会有如下信息（这里省略了大部分无关信息）：

```java
Constant pool:
 
.........
.........
 
   #5 = Utf8               computer
   #6 = Utf8               Lcom/jg/zhang/Computer;
 
.........
.........
 
{
 
  private com.jg.zhang.Computer computer;
    flags: ACC_PRIVATE
 
.........
.........
 
}
```

从反编译的结果可以看出， 源文件中定义了一个Computer类型的字段computer， 并且是private的。 然后常量池中有这个字段的字段名和描述符。 其中常量池第五项的CONSTANT_Utf8_info是字段名， 第六项的CONSTANT_Utf8_info是该字段的描述符。这里有一点需要说明， 在反编译Programer.class时，由于computer是私有的， 要加- private选项， 否则的话， 虽然常量池中有字段引用信息， 但是不会输出字段信息， 即下面这两行不会输出：

```java
  private com.jg.zhang.Computer computer;
    flags: ACC_PRIVATE
```

如果在javap中加入 - private选项， 那么就会有上面两行的输出。 使用的命令如下：

```java
javap -c -v -private -classpath . com.jg.zhang.Programer
```

根据反编译的结果，可以下面给出示意图， 该图说明了与computer相对应的field_info是不合引用常量池的 （ 其中虚线范围内表示常量池）：

<a href="/img/post/jvm/24.png" target="_blank"><img src="/img/post/jvm/24.png" /></a>

### class文件中的methods_count和methods

fields下面的信息是methods_count和methods 。 methods_count描述的是当前的类中定义的方法的个数， 注意， 这里包括静态方法， 但不包括从父类继承的方法。 如果当前class文件是由一个接口生成的， 那么这里的methods_count描述的是接口中定义的抽象方法的数量， 我们知道， 接口中定义的方法默认都是公有的。此外需要说明的是， 编译器可能会在编译时向class文件增加额外的方法， 也就是说， class文件中的方法的数量可能多于源文件中由用户定义的方法。 举例来说： 如果当前类没有定义构造方法， 那么编译器会增加一个无参数的构造函数<init>； 如果当前类或接口中定义了静态变量， 并且使用初始化表达式为其赋值， 或者定义了static静态代码块， 那么编译器在编译的时候会默认增加一个静态初始化方法<clinit> 。 

位于methods_count下面的数据叫做methods ， 可以把它看做一个数组， 数组中的每一项是一个method_info 。这个数组中一共有methods_count个method_info ， 每个method_info 都是对一个方法的描述。 下面我们详细讲解method_info 的结构。 每个method_info 的结构如下， 几乎和field_info的结构是一样的：

<a href="/img/post/jvm/25.png" target="_blank"><img src="/img/post/jvm/25.png" /></a>

####（1）access_flags

其中access_flags占两个字节， 描述的是方法的访问标志信息。 这里就不在详细介绍了， 下面给出一张表格（该表格来自《深入Java虚拟机》）：

| 标志位名称       | 标记值  | 设定含义            | 设定者   |
| --------         | :-----  | :----               | :----    |
| ACC_PUBLIC       | 0x0001  | 方法设为public      | 类和接口 |
| ACC_PRIVATE      | 0x0002  | 方法设为private     | 类       |
| ACC_PROTECTED    | 0x0004  | 方法设为protected   | 类       |
| ACC_STATIC       | 0x0008  | 方法设为static      | 类       |
| ACC_FINAL        | 0x0010  | 方法设为final       | 类       |
| ACC_SYNCHRONIZED | 0x0020  | 方法设为sychronized | 类       |
| ACC_NATIVE       | 0x0100  | 方法设为native      | 类       |
| ACC_ABSTRACT     | 0x0400  | 方法设为abstract    | 类和接口 |
| ACC_STRICT       | 0x0800  | 方法设为strictFP    | 类和接口的<clinit>方法 |

####（2）name_index
access_flags下面的两个字节是name_index， 这是一个指向常量池的索引， 它描述的是当前方法的方法名。 这个索引指向常量池中的一个CONSTANT_Utf8_info数据项。 这个CONSTANT_Utf8_info数据项中存放的字符串就是当前方法的方法名。 


####（3）descriptor_index
name_index下面的两个字节叫做descriptor_index ， 它同样是一个指向常量池的索引， 它描述的是当前方法的描述符。 这个索引指向常量池中的一个CONSTANT_Utf8_info数据项。 这个CONSTANT_Utf8_info数据项中存放的字符串就是当前方法的描述符（关于方法描述符， 在前面的博客中已经有过详细的讲解， 如果不明白， 请参考前面的博客： 深入理解Java Class文件格式（二））。 


####（4）attributes_count和attributes
descriptor_index 下面是attributes_count和attributes 。 这是对当前方法所具有的属性的描述。 这里的属性和源文件中的属性不是同一个概念， 在源文件测层面中， 属性是字段的另一种叫法， 希望读者不要疑惑。读者也不要轻视class文件中的属性， 这些属性可以描述很多的信息。 我们会在后面的文章中进行介绍。 

attributes_count表示这个字段有几个属性。attributes 可以看成一个数组， 数组中的每一项都是一个attribute_info ， 每个attribute_info 表示一个属性， 数组中一共有attributes_count个属性。可以出现在method_info 中的属性有三种， 分别是Code， Deprecated， Exceptions 和Synthetic。 在这几个属性中， 尤其是Code和Exceptions 非常重要， 这两个属性对于在class文件中完整描述一个方法起着至关重要的作用， 其中Code属性中存放方法的字节面指令，Exceptions 属性是对方法声明中抛出的异常的描述 。 这两属性以及其他一些属性， 会在下一篇文章中详细介绍， 敬请关注。


介绍完了每个method_info的结构， 下面我们以代码来说明， 还是使用上面的源码：

```java
package com.jg.zhang;
 
public class Programer extends Person{
 
	
	private Computer computer;
	
	public Programer(Computer computer){
		this.computer = computer;
	}
	
	public void doWork(){
		computer.calculate();
	}
}
```

反编译之后， 常量池中会有如下信息（这里省略了大部分无关信息）：

```java

Constant pool:
 
.........
 
   #7 = Utf8               <init>
   #8 = Utf8               (Lcom/jg/zhang/Computer;)V
 
.........
 
  #12 = Utf8               ()V
 
.........
 
  #19 = Utf8               doWork
 
{
 
.........
 
  public com.jg.zhang.Programer(com.jg.zhang.Computer);
    flags: ACC_PUBLIC
 
.........
 
  public void doWork();
    flags: ACC_PUBLIC
 
.........
}
```

由反编译结果可以看出， 该类中定义了两个方法， 其中一个是构造方法， 一个是doWork方法， 且这两个方法都是public的。 这两个方法的描述信息都存放在常量池。 其中第7项的CONSTANT_Utf8_info为构造方法的方法名， 第8项的CONSTANT_Utf8_info为构造方法的方法描述符， 第19项的CONSTANT_Utf8_info为doWork方法的方法名， 第12项的CONSTANT_Utf8_info为doWork方法的方法描述符。 

根据常量池中的信息， 可以得出如下的示意图， 该示意图形象的说明了class文件中的method_info是如何引用常量池中的数据项来描述当前类中定义的方法的。 图中虚线范围内表示常量池所在的区域：

<a href="/img/post/jvm/25.png" target="_blank"><img src="/img/post/jvm/25.png" /></a>

### 总结

到此为止， 我们就介绍完了class文件中的fields和methods， 进行一下总结。 

fields是对当前类中定义的字段的描述， 其中每个字段使用一个field_info表示， fields中有fields_count个field_info。

methods是对当前类或者接口中声明的方法的描述， 其中每个方法使用一个method_info表示， methods中有methods_count个method_info。 
