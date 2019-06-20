---
layout: post
title:  "深入理解Java Class文件格式(三)"
date:   2019-06-07 20:19:20 +0800
categories: Jvm
tags: Jvm
---

首先， 让我们回顾一下关于class文件格式的之前两篇博客的主要内容。 在 深入理解Java Class文件格式（一） 中， 讲解了class文件在整个java体系结构中的位置和作用， 讲解了class文件中的魔数和版本号相关的信息， 并且对常量池进行了概述。 在 深入理解Java Class文件格式（二） 中， 主要讲解了class文件中的特殊字符串， 包括类的全限定名， 字段描述符和方法描述符， 这些特殊字符串大量出现在class文件的常量池中， 是理解常量池的基础。 本文会详细讲解常量池中的各个数据项。

如果你还没有读过前两篇文章， 建议先去读一下， 这样才能保持知识的连贯性。 前两篇文章的链接已经在上面给出。 下面开始讲解常量池。

### 常量池中各数据项类型详解

关于常量池的大概内容， 已经在 深入理解Java Class文件格式（一） 中讲解过了， 这篇文章中还介绍了常量池中的11种数据类型。 本文的任务是详细讲解这11种数据类型， 深度剖析源文件中的各种信息是以什么方式存放在常量池中的。 

我们知道， 常量池中的数据项是通过索引来引用的， 常量池中的各个数据项之间也会相互引用。在这11中常量池数据项类型中， 有两种比较基础， 之所以说它们基础， 是因为这两种类型的数据项会被其他类型的数据项引用。 这两种数据类型就是CONSTANT_Utf8 和 CONSTANT_NameAndType ， 其中CONSTANT_NameAndType类型的数据项（CONSTANT_NameAndType_info）也会引用CONSTANT_Utf8类型的数据项（CONSTANT_Utf8_info） 。 与其他介绍常量池的书籍或其他资料不同， 本着循序渐进和先后分明的原则， 我们首先对这两种比较基本的类型的数据项进行介绍， 然后再依次介绍其他9中数据项。 

#### （1） CONSTANT_Utf8_info

一个CONSTANT_Utf8_info是一个CONSTANT_Utf8类型的常量池数据项， 它存储的是一个常量字符串。 常量池中的所有字面量几乎都是通过CONSTANT_Utf8_info描述的。下面我们首先讲解CONSTANT_Utf8_info数据项的存储格式。在前面的文章中， 我们提到， 常量池中数据项的类型由一个整型的标志值（tag）决定， 所以所有常量池类型的info中都必须有一个tag信息， 并且这个tag值位于数据项的第一个字节上。 一个11中常量池数据类型， 所以就有11个tag值表示这11中类型。而CONSTANT_Utf8_info的tag值为1， 也就是说如果虚拟机要解析一个常量池数据项， 首先去读这个数据项的第一个字节的tag值， 如果这个tag值为1， 那么就说明这个数据项是一个CONSTANT_Utf8类型的数据项。 紧挨着tag值的两个字节是存储的字符串的长度length， 剩下的字节就存储着字符串。 所以， 它的格式是这样的：

<a href="/img/post/jvm/5.png" target="_blank"><img src="/img/post/jvm/5.png" /></a>

其中tag占一个字节， length占2个字节， bytes代表存储的字符串， 占length字节。所以， 如果这个CONSTANT_Utf8_info存储的是字符串"Hello"， 那么他的存储形式是这样的：

<a href="/img/post/jvm/6.png" target="_blank"><img src="/img/post/jvm/6.png" /></a>

现在我们知道了CONSTANT_Utf8_info数据项的存储形式， 那么CONSTANT_Utf8_info数据项都存储了什么字符串呢？ CONSTANT_Utf8_info可包括的字符串主要以下这些：

- 程序中的字符串常量

- 常量池所在当前类（包括接口和枚举）的全限定名

- 常量池所在当前类的直接父类的全限定名

- 常量池所在当前类型所实现或继承的所有接口的全限定名

- 常量池所在当前类型中所定义的字段的名称和描述符

- 常量池所在当前类型中所定义的方法的名称和描述符

- 由当前类所引用的类型的全限定名

- 由当前类所引用的其他类中的字段的名称和描述符

- 由当前类所引用的其他类中的方法的名称和描述符

- 与当前class文件中的属性相关的字符串， 如属性名等

总结一下， 其中有这么五类： 程序中的字符串常量， 类型的全限定名， 方法和字段的名称， 方法和字段的描述符， 属性相关字符串。 程序中的字符串常量不用多说了， 我们经常使用它们创建字符串对象， 属性相关的字符串， 等到讲到class中的属性信息（attibute）时自会提及。 方法和字段的名称也不用多说了 。 剩下的就是类型的全限定名，方法和字段的描述符， 这就是上篇文章中提及的"特殊字符串"， 不熟悉的同学可以先读一下上篇文章 深入理解Java Class文件格式（二） 。 还有一点需要说明， 类型的全限定名， 方法和字段的名称， 方法和字段的描述符， 可以是本类型中定义的， 也可能是本类中引用的其他类的。

下面我们通过一个例子来进行说明。 示例源码：

```java
package com.jg.zhang;
 
public class Programer extends Person {
 
    static String company = "CompanyA";
    
    static{
        System.out.println("staitc init");
    }
    
    
    String position;
    Computer computer;
 
    public Programer() {
        this.position = "engineer";
        this.computer = new Computer();
    }
    
    public void working(){
        System.out.println("coding...");
        computer.working();
    }
}
```

别看这个类简单， 但是反编译后， 它的常量池有53项之多。 在这53项常量池数据项中， 各种类型的数据项都有， 当然也包括不少的CONSTANT_Utf8_info 。 下面只列出反编译后常量池中的CONSTANT_Utf8_info 数据项：

```java
#2 = Utf8               com/jg/zhang/Programer          //当前类的全限定名
#4 = Utf8               com/jg/zhang/Person             //父类的全限定名
#5 = Utf8               company                         //company字段的名称
#6 = Utf8               Ljava/lang/String;              //company和position字段的描述符
#7 = Utf8               position                        //position字段的名称
#8 = Utf8               computer                        //computer字段的名称
#9 = Utf8               Lcom/jg/zhang/Computer;         //computer字段的描述符
#10 = Utf8              <clinit>                        //类初始化方法（即静态初始化块）的方法名
#11 = Utf8              ()V                             //working方法的描述符
#12 = Utf8              Code                            //Code属性的属性名
#14 = Utf8              CompanyA                        //程序中的常量字符串
#19 = Utf8              java/lang/System                //所引用的System类的全限定名
#21 = Utf8              out                             //所引用的out字段的字段名
#22 = Utf8              Ljava/io/PrintStream;           //所引用的out字段的描述符
#24 = Utf8              staitc init                     //程序中的常量字符串
#27 = Utf8              java/io/PrintStream             //所引用的PrintStream类的全限定名
#29 = Utf8              println                         //所引用的println方法的方法名
#30 = Utf8              (Ljava/lang/String;)V           //所引用的println方法的描述符
#31 = Utf8              LineNumberTable                 //LineNumberTable属性的属性名
#32 = Utf8              LocalVariableTable              //LocalVariableTable属性的属性名
#33 = Utf8              <init>                          //当前类的构造方法的方法名
#41 = Utf8              com/jg/zhang/Computer           //所引用的Computer类的全限定名
#45 = Utf8              this                            //局部变量this的变量名
#46 = Utf8              Lcom/jg/zhang/Programer;        //局部变量this的描述符
#47 = Utf8              working                         //woking方法的方法名
#49 = Utf8              coding...                       //程序中的字符串常量
#52 = Utf8              SourceFile                      //SourceFile属性的属性名
#53 = Utf8              Programer.java                  //当前类所在的源文件的文件名
```

这里要强调一下， 源文件中的几乎所有可见的字符串都存放在CONSTANT_Utf8_info中， 其他类型的常量池项只不过是对CONSTANT_Utf8_info的引用。 其他常量池项， 把引用的CONSTANT_Utf8_info组合起来， 进而可以描述更多的信息。 下面将要介绍的CONSTANT_NameAndType_info就可以验证这个结论。

####（2） CONSTANT_NameAndType类型的数据项

常量池中的一个CONSTANT_NameAndType_info数据项， 可以看做CONSTANT_NameAndType类型的一个实例 。 从这个数据项的名称可以看出， 它描述了两种信息，第一种信息是名称（Name）， 第二种信息是类型（Type） 。 这里的名称是指方法的名称或者字段的名称， 而Type是广义上的类型， 它其实描述的是字段的描述符或方法的描述符。 也就是说， 如果Name部分是一个字段名称， 那么Type部分就是相应字段的描述符； 如果Name部分描述的是一个方法的名称， 那么Type部分就是对应的方法的描述符。 也就是说， 一个CONSTANT_NameAndType_info就表示了一个方法或一个字段。 

下面先看一下CONSTANT_NameAndType_info数据项的存储格式。 既然是常量池中的一种数据项类型， 那么它的第一个字节也是tag， 它的tag值是12， 也就是说， 当虚拟机读到一个tag为12的常量池数据项， 就可以确定这个数据项是一个CONSTANT_NameAndType_info 。 tag值一下的两个字节叫做name_index， 它指向常量池中的一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info中存储的就是方法或字段的名称。 name_index以后的两个字节叫做descriptor_index， 它指向常量池中的一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info中存储的就是方法或字段的描述符。 下图表示它的存储布局：

<a href="/img/post/jvm/7.png" target="_blank"><img src="/img/post/jvm/7.png" /></a>

下面举一个实例进行说明， 实例的源码为：

```java
package com.jg.zhang;
 
public class Person {
 
	int age;
 
	int getAge(){
		return age;
	}
}
```

这个Person类很简单， 只有一个字段age， 和一个方法getAge 。 将这段代码使用javap工具反编译之后， 常量池信息如下：

```java
   #1 = Class              #2             //  com/jg/zhang/Person
   #2 = Utf8               com/jg/zhang/Person
   #3 = Class              #4             //  java/lang/Object
   #4 = Utf8               java/lang/Object
   #5 = Utf8               age
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Methodref          #3.#11         //  java/lang/Object."<init>":()V
  #11 = NameAndType        #7:#8          //  "<init>":()V
  #12 = Utf8               LineNumberTable
  #13 = Utf8               LocalVariableTable
  #14 = Utf8               this
  #15 = Utf8               Lcom/jg/zhang/Person;
  #16 = Utf8               getAge
  #17 = Utf8               ()I
  #18 = Fieldref           #1.#19         //  com/jg/zhang/Person.age:I
  #19 = NameAndType        #5:#6          //  age:I
  #20 = Utf8               SourceFile
  #21 = Utf8               Person.java
```

常量池一共有21项， 我们可以看到， 一共有两个CONSTANT_NameAndType_info 数据项， 分别是第#11项和第#19项， 其中第#11项的CONSTANT_NameAndType_info又引用了常量池中的第#7项和第#8项， 被引用的这两项都是CONSTANT_Utf8_info ， 它们中存储的字符串常量值分别是 <init> 和 （）V。 其实他们加起来表示的就是父类Object的构造方法。 那么这里为什么会是父类Object的构造方法而不是本类的构造方法呢？ 这是因为类中定义的方法如果不被引用（也就是说在当前类中不被调用）， 那么常量池中是不会有相应的 CONSTANT_NameAndType_info 与之对应的， 只有引用了一个方法， 才有相应的CONSTANT_NameAndType_info 与之对应。 这也是为什么说CONSTANT_NameAndType_info 是方法的符号引用的一部分的原因。 （这里提到一个新的概念， 叫做方法的符号引用， 这个概念会在后面的博客中进行讲解） 可以看到， 在源码存在两个方法， 分别是编译器默认添加的构造方法和我们自己定义的getAge方法， 因为并没有在源码中显示的调用这两个方法，所以在常量池中并不存在和这两个方法相对应的CONSTANT_NameAndType_info 。  之所以会存在父类Object的构造方法对应的CONSTANT_NameAndType_info ， 是因为子类构造方法中会默认调用父类的无参数构造方法。 我们将常量中的其他信息去掉， 可以看得更直观：

<a href="/img/post/jvm/8.png" target="_blank"><img src="/img/post/jvm/8.png" /></a>

下面讲解常量池第#19项的CONSTANT_NameAndType_info ， 它引用了常量池第#5项和第#6项， 这两项也是CONSTANT_Utf8_info 项， 其中存储的字符串分别是age和I， 其中age是源码中字段age的字段名， I是age字段的描述符。 所以这个CONSTANT_NameAndType_info 就表示对本类中的字段age的引用。 除去常量池中的其他信息， 可以看得更直观：

<a href="/img/post/jvm/9.png" target="_blank"><img src="/img/post/jvm/9.png" /></a>

和方法相同， 只定义一个字段而不引用它（在源码中表现为不访问这个变量）， 那么在常量池中也不会存在和该字段相对应的CONSTANT_NameAndType_info 项。这也是为什么说CONSTANT_NameAndType_info作为字段符号引用的一部分的原因。 （这里提到一个新的概念， 叫做字段的符号引用， 这个概念会在后面的博客中进行讲解） 在本例中之所以会出现这个CONSTANT_NameAndType_info ， 是因为在源码的getAge方法中访问了这个字段：

```java
int getAge(){
		return age;
}
```

下面给出这两个CONSTANT_NameAndType_info真实的内存布局图：

和Object构造方法相关的CONSTANT_NameAndType_info的示意图：

<a href="/img/post/jvm/10.png" target="_blank"><img src="/img/post/jvm/10.png" /></a>

和age字段相关的CONSTANT_NameAndType_info示意图：

<a href="/img/post/jvm/11.png" target="_blank"><img src="/img/post/jvm/11.png" /></a>

这两张图能够很好的反映出CONSTANT_NameAndType_info和CONSTANT_Utf8_info 这两种常量池数据项的数据存储方式， 也能够真实的反应CONSTANT_NameAndType_info和CONSTANT_Utf8_info 的引用关系。 

### 总结

本篇博客就到此为止， 在本文中我们主要介绍了常量池中的两种数据项： CONSTANT_NameAndType_info 和 CONSTANT_Utf8_info  。 其中CONSTANT_Utf8_info存储的是源文件中的各种字符串， 而CONSTANT_NameAndType_info表述的是源文件中对一个字段或方法的符号引用的一部分（即 方法名加方法描述符， 或者是 字段名加字段描述符）。在下一篇博客中， 继续讲解常量池中的其他类型的数据项 。 
