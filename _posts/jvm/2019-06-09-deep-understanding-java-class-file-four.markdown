---
layout: post
title:  "深入理解Java Class文件格式(四)"
date:   2019-06-09 20:19:20 +0800
categories: Jvm
tags: Jvm
---

### 前情回顾

在上一篇博客深入理解Java Class文件格式（三） 中， 介绍了常量池中的两种类型的数据项， 分别是

1.CONSTANT_Utf8_info

2.CONSTANT_NameAndType_info 。

CONSTANT_Utf8_info中存储了几乎所有类型的字符串， 包括方法名， 字段名， 描述符等等。 而CONSTANT_NameAndType_info是方法符号引用或字段的符号引用的一部分， 也就是说， 如果在源文件中调用了一个方法， 或者引用了一个字段（不管是本类中的方法和字段， 还是引用其他类中的方法和字段）， 那么和这个方法或在字段相对应的CONSTANT_NameAndType_info 就会出现在常量池中。 注意， 只有引用了一个方法或字段， 常量池中才会存在和它对应的CONSTANT_NameAndType_info ， 如果只在当前类中定义了一个字段而不访问它， 或者定义了一个方法而不调用它， 那么常量池中就不会出现对应的CONSTANT_NameAndType_info 数据项。 CONSTANT_NameAndType_info 中引用了两个CONSTANT_Utf8_info， 一个叫做name_index, 存储方法名或字段名， 一个叫做descriptor_index, 存储方法描述符或字段描述符。 

下面我们继续详细讲解常量池中的其他类型的数据项， 本文接着前几篇文章写的， 建议先读前几篇文章。   

常量池中各数据项类型详解（续）

###（3）CONSTANT_Integer_info

一个常量池中的CONSTANT_Integer_info数据项, 可以看做是CONSTANT_Integer类型的一个实例。 它存储的是源文件中出现的int型数据的值。 同样， 作为常量池中的一种数据类型， 它的第一个字节也是一个tag值， 它的tag值为3， 也就是说， 当虚拟机读到一个tag值为3的数据项时， 就知道这个数据项是一个CONSTANT_Integer_info， 它存储的是int型数值的值。 紧挨着tag的下面4个字节叫做bytes， 就是int型数值的整型值。 它的内存布局如下：

<a href="/img/post/jvm/12.png" target="_blank"><img src="/img/post/jvm/12.png" /></a>

下面以示例代码进行说明， 示例代码如下：

```java
package com.jg.zhang;
 
public class TestInt {
	
	void printInt(){
		System.out.println(65535);
	}
}
```

将上面的类生成的class文件反编译：

```java

  ..................
  ..................


Constant pool:


   ..................
   ..................


  #21 = Integer            65535

   ..................
   ..................

{


     ..................
     ..................

  void printInt();
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #15                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #21                 // int 65535
         5: invokevirtual #22                 // Method java/io/PrintStream.println:(I)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0       9     0  this   Lcom/jg/zhang/TestInt;
}
```

上面的输出结果中， 保留了printInt方法的反编译结果， 并且保留了常量池中的第21项。 首先看printInt方法反编译结果中的索引为3 的字节码指令：

```java
3: ldc           #21                 // int 65535
```

这条ldc指令， 引用了常量池中的第21项， 而第21项是一个CONSTANT_Integer_info， 并且这个CONSTANT_Integer_info存储的整型值为65535 。 

### （4）CONSTANT_Float_info

一个常量池中的CONSTANT_Float_info数据项, 可以看做是CONSTANT_Float类型的一个实例。 它存储的是源文件中出现的float型数据的值。 同样， 作为常量池中的一种数据类型， 它的第一个字节也是一个tag值， 它的tag值为4， 也就是说， 当虚拟机读到一个tag值为4的数据项时， 就知道这个数据项是一个CONSTANT_Float_info， 并且知道它存储的是float型数值。 紧挨着tag的下面4个字节叫做bytes， 就是float型的数值。 它的内存布局如下：

<a href="/img/post/jvm/12.png" target="_blank"><img src="/img/post/jvm/12.png" /></a>

举例说明， 如果源文件中的一句代码使用了一个float值， 如下所示：

```java
void printFloat(){
		System.out.println(1234.5f);
}
```

那么在这个类的常量池中就会有一个CONSTANT_Float_info与之相对应， 这个CONSTANT_Float_info的形式如下：

```java
代码反编译结果如下：
Constant pool:

.............
.............

  #29 = Float              1234.5f

............
............

{

............
............

  void printFloat();
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #15                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #29                 // float 1234.5f
         5: invokevirtual #30                 // Method java/io/PrintStream.println:(F)V
         8: return
      LineNumberTable:
        line 10: 0
        line 11: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0       9     0  this   Lcom/jg/zhang/TestInt;
}
```

### （5）CONSTANT_Long_info

一个常量池中的CONSTANT_Long_info数据项, 可以看做是CONSTANT_Long类型的一个实例。 它存储的是源文件中出现的long型数据的值。 同样， 作为常量池中的一种数据类型， 它的第一个字节也是一个tag值， 它的tag值为5， 也就是说， 当虚拟机读到一个tag值为5的数据项时， 就知道这个数据项是一个CONSTANT_Long_info， 并且知道它存储的是long型数值。 紧挨着tag的下面8个字节叫做bytes， 就是long型的数值。 它的内存布局如下：

<a href="/img/post/jvm/13.png" target="_blank"><img src="/img/post/jvm/13.png" /></a>

举例说明， 如果源文件中的一句代码使用了一个long型的数值， 如下所示：

```java
void printLong(){
		System.out.println(123456L);
}
```

那么在这个类的常量池中就会有一个CONSTANT_Long_info与之相对应， 这个CONSTANT_Long_info的形式如下：

```java
Constant pool:

..............
..............

  #21 = Long               123456l

..............
..............

{

..............
..............

  void printLong();
    flags:
    Code:
      stack=3, locals=1, args_size=1
         0: getstatic     #15                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc2_w        #21                 // long 123456l
         6: invokevirtual #23                 // Method java/io/PrintStream.println:(J)V
         9: return
      LineNumberTable:
        line 7: 0
        line 8: 9
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      10     0  this   Lcom/jg/zhang/TestInt;
}
```

### （6）CONSTANT_Double_info

一个常量池中的CONSTANT_Double_info数据项, 可以看做是CONSTANT_Double类型的一个实例。 它存储的是源文件中出现的double型数据的值。 同样， 作为常量池中的一种数据类型， 它的第一个字节也是一个tag值， 它的tag值为6， 也就是说， 当虚拟机读到一个tag值为6的数据项时， 就知道这个数据项是一个CONSTANT_Double_info， 并且知道它存储的是double型数值。 紧挨着tag的下面8个字节叫做bytes， 就是double型的数值。 它的内存布局如下：

<a href="/img/post/jvm/13.png" target="_blank"><img src="/img/post/jvm/13.png" /></a>

举例说明， 如果源文件中的一句代码使用了一个double型的数值， 如下所示：

```java
void printDouble(){
		System.out.println(123456D);
}
```

那么在这个类的常量池中就会有一个CONSTANT_Double_info与之相对应， 这个CONSTANT_Double_info的形式如下：

```java
Constant pool:

..............
..............

  #21 = Double             123456.0d

..............
..............

{

..............
..............

  void printDouble();
    flags:
    Code:
      stack=3, locals=1, args_size=1
         0: getstatic     #15                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc2_w        #21                 // double 123456.0d
         6: invokevirtual #23                 // Method java/io/PrintStream.println:(D)V
         9: return
      LineNumberTable:
        line 7: 0
        line 8: 9
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      10     0  this   Lcom/jg/zhang/TestInt;
}
```

### (7) CONSTANT_String_info

在常量池中， 一个CONSTANT_String_info数据项， 是CONSTANT_String类型的一个实例。 它的作用是存储文字字符串， 可以把他看做是一个存在于class文件中的字符串对象。 同样， 它的第一个字节是tag值， 值为8 ， 也就是说， 虚拟机访问一个数据项时， 判断tag值为8 ， 就说明访问的数据项是一个CONSTANT_String_info 。 紧挨着tag的后两个字节是一个叫做string_index的常量池引用， 它指向一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info存放的才是字符串的字面量。 它的内存布局如下：

<a href="/img/post/jvm/14.png" target="_blank"><img src="/img/post/jvm/14.png" /></a>

举例说明， 如果源文件中的一句代码使用了一个字符串常量， 如下所示：

```java
void printStrng(){
		System.out.println("abcdef");
}
```

那么在这个类的常量池中就会有一个CONSTANT_String_info与之相对应， 反编译结果如下：

```java
Constant pool:

..............
..............
  
  #21 = String             #22            //  abcdef
  #22 = Utf8               abcdef

..............
..............

{

..............
..............

  void printStrng();
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #15                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #21                 // String abcdef
         5: invokevirtual #23                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 7: 0
        line 8: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0       9     0  this   Lcom/jg/zhang/TestInt;
}
```

其中printString方法中索引为3的字节码指令ldc引用常量池中的第21项， 第21项是一个CONSTANT_String_info， 这个位于第21项的CONSTANT_String_info又引用了常量池的第22项， 第22项是一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info中存储的字符串是 abcdef 。 引用关系的内存布局如下：

<a href="/img/post/jvm/15.png" target="_blank"><img src="/img/post/jvm/15.png" /></a>

### 总结

本文就到此为止。 最后总结一下， 本文主要讲解了常量池中的五中数据项， 分别为CONSTANT_Integer_info， CONSTANT_Float_info， CONSTANT_Long_info， CONSTANT_Double_info 和CONSTANT_String_info 。 这几种常量池数据项都是直接存储的常量值，而不是符号引用。 这里又一次出现了符号引用的概念， 这个概念将会在下一篇博客中详细讲解， 因为下一篇博客要介绍的剩下的四种常量池数据项， 都是符号引用， 这四种表示符号引用的数据项又会直接或间接引用上篇文章中介绍的CONSTANT_NameAndType_info和CONSTANT_Utf8_info， 所以说CONSTANT_NameAndType_info是符号引用的一部分。 

从本文中我们还可以知道。 虽然说CONSTANT_String_info是直接存储值的数据项， 但是CONSTANT_String_info有点特别， 因为它不是直接存储字符串， 而是引用了一个CONSTANT_Utf8_info， 这个被引用的CONSTANT_Utf8_info中存储了字符串。 

最后， 列出下一篇博文要介绍的剩下的四种常量池数据项：

CONSTANT_Class_info

CONSTANT_Fieldref_info

CONSTANT_Methodref_info

CONSTANT_InterfaceMethodref_info
