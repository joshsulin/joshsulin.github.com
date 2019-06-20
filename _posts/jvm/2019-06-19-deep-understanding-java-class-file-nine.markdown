---
layout: post
title:  "深入理解Java Class文件格式(九)"
date:   2019-06-19 20:19:20 +0800
categories: Jvm
tags: Jvm
---

经过前八篇关于class文件的博客，关于class文件格式的内容也基本上写完了。本文是关于class文件格式的最后一篇。在这篇博客中，将会讲解关于方法的几个属性。理解这篇博客的内容，对于理解JVM执行引擎起着重要作用。

在前面几篇博客中，我们知道在class文件中描述一个方法，会使用一个method_info。这个method_info中存放了方法的修饰符标志位，还引用了常量池中的项，这些常量池数据项描述了在当前类中定义的某个方法的方法名，方法的描述符。关于这部分的内容，请参考我之前的博客：深入理解Java Class文件格式（七）。 

但是method_info中并没有存放方法的字节码，也就是指令。我们知道，对于一个方法来说，只要它不是抽象的（抽象类中的抽象方法或者接口中的方法），那么肯定就会存在指令。那么这些指令存放在哪里呢？还有，方法中的异常处理器（try-catch块）是如何在class文件中表述的？方法声明抛出的异常是如何描述的呢？如果你对这几个问题感兴趣， 或许你会在这篇博客中找到答案， 或者受到一些启发。

为了知识的连贯性，我们首先简单回顾一下method_info的结构，因为method_info与本文有着密切的关系。method_info 的结构如下：

<a href="/img/post/jvm/37.png" target="_blank"><img src="/img/post/jvm/37.png" /></a>

深入理解Java Class文件格式（七）这篇博客中已经讲解过access_flags，name_index，descriptor_index。他们分别描述方法的访问修饰符，方法名和方法描述符。从上图可以看出，method_info中还有attributes_count和attributes。也就是说每个方法可以有另个或多个属性。本文要讲解的方法中的字节码指令，异常处理器和方法声明抛出的异常，都存放在这些属性中。

### Code属性

code属性是方法的一个最重要的属性。因为它里面存放的是方法的字节码指令，除此之外还存放了和操作数栈，局部变量相关的信息。所有不是抽象的方法，都必须在method_info中的attributes中有一个Code属性。下面是Code属性的结构，为了更直观的展示Code属性和method_info的包含关系，特意画出了method_info：

<a href="/img/post/jvm/38.png" target="_blank"><img src="/img/post/jvm/38.png" /></a>

下面依次介绍code属性中的各个部分。

和上一篇博客中介绍的其他属性一样，attribute_name_index指向常量池中的一个CONSTANT_Utf8_info ，这个CONSTANT_Utf8_info中存放的是当前属性的名字"Code"。

attribute_length给出了当前Code属性的长度（不包括前六字节）。

max_stack 指定当前方法被执行引擎执行的时候，在栈帧中需要分配的操作数栈的大小。

max_locals指定当前方法被执行引擎执行的时候，在栈帧中需要分配的局部表量表的大小。注意，这个数字并不是局部变量的个数，因为根据局部变量的作用域不同，在执行到一个局部变量以外时，下一个局部变量可以重用上一个局部变量的空间（每个局部变量在局部变量表中占用一个或两个Slot）。方法中的局部变量包括方法的参数，方法的默认参数this，方法体中定义的变量，catch语句中的异常对象。关于执行引擎的相关内容会在后面的博客中讲到。

code_length指定该方法的字节码的长度，class文件中每条字节码占一个字节。

code存放字节码指令本身，它的长度是code_length个字节。

exception_table_length指定异常表的大小

exception_table就是所谓的异常表，它是对方法体中try-catch_finally的描述。exception_table可以看做是一个数组，每个数组项是一个exception_info结构，一般来说每个catch块对应一个exception_info，编译器也可能会为当前方法生成一些exception_info。 exception_info的结构如下（为了直观的显示exception_info，exception_table和Code属性的关系， 画出了Code属性，的话读者就会更清楚各个数据项之间的位置关系和包含关系）：

<a href="/img/post/jvm/39.png" target="_blank"><img src="/img/post/jvm/39.png" /></a>

下面讲解exception_info中的各个字段的意思。

start_pc是从字节码（Code属性中的code部分）起始处到当前异常处理器起始处的偏移量。

end_pc是从字节码起始处到当前异常处理器末尾的偏移量。

handler_pc是指当前异常处理器用来处理异常（即catch块）的第一条指令相对于字节码开始处的偏移量。

catch_type是一个常量池索引，指向常量池中的一个CONSTANT_Class_info数据项，该数据项描述了catch块中的异常的类型信息。这个类型必须是java.lang.Throwable的或其子类。

所以可以总结，一个异常处理器（exception_info）的意思是：如果偏移量从start_pc到end_pc之间的字节码出现了catch_type描述的类型的异常，那么就跳转到偏移量为handler_pc的字节码处去执行。如果catch_type为0，就代表不引用任何常量池项（再回顾一下，常量池中的项是从1开始计的），那么这个exception_info用于实现finally子句。

我们一直在介绍Code属性，只不过刚才进行了一个小插曲，介绍了Code属性中的exception_table中的exception_info的详细信息。 下面我们继续介绍Code 属性中的其他信息， 希望读者不要被绕晕了 : )

attributes_count 表示当前Code属性中存在的其他属性的个数。现在我们知道，class中的属性，不仅会出现在顶层的class中，会存在field_info中，会存在method_info中，甚至还会出现在属性中。

attributes可以看做是一个数组， 里面存放了Code属性中的其他属性。 Code 属性中可以出现的其他属性有LineNumberTable和LocalVariableTable 。 这两个属性会在下面介绍。

#### LineNumberTable属性

LineNumberTable属性存在于Code属性中，它建立了字节码偏移量到源代码行号之间的联系。这个属性是可选的，编译器可以选择不生成该属性。下面是该属性的结构（同样给出了全局的位置关系，LineNumberTable在图的右下角部分）：

<a href="/img/post/jvm/40.png" target="_blank"><img src="/img/post/jvm/40.png" /></a>

由于这个属性并不是重点， 我们在此简单的讲述。 

每个LineNumberTable中的line_number_table部分，可以看做是一个数组，数组的每项是一个line_number_info，每个line_number_info结构描述了一条字节码和源码行号的对应关系。其中start_pc是这个line_number_info描述的字节码指令的偏移量，line_number是这个line_number_info描述的字节码指令对应的源码中的行号。可以看出，方法中的每条字节码都对应一个line_number_info，这些line_number_info中的line_number可以指向相同的行号，因为一行源码可以编译出多条字节码。

#### LocalVariableTable属性 

LocalVariableTable属性建立了方法中的局部变量与源代码中的局部变量之间的对应关系。这个属性存在于Code属性中。这个属性是可选的，编译器可以选择不生成这个属性。该属性的结构如下：（同样给出了全局的位置关系图，LocalVariableTable 在该图的右下角 ）

<a href="/img/post/jvm/41.png" target="_blank"><img src="/img/post/jvm/41.png" /></a>

由于这个属性相对不那么重要，这里只是大概讲解一下。

每个LocalVariableTable的local_variable_table部分可以看做是一个数组，每个数组项是一个叫做local_variable_info的结构，该结构描述了某个局部变量的变量名和描述符，还有和源代码的对应关系。下面讲解local_variable_info的各个部分：

start_pc是当前local_variable_info所对应的局部变量的作用域的起始字节码偏移量；

length是当前local_variable_info所对应的局部变量的作用域的大小。也就是从字节码偏移量start_pc到start_pc+length就是当前局部变量的作用域范围；

name_index指向常量池中的一个CONSTANT_Utf8_info，该CONSTANT_Utf8_info描述了当前局部变量的变量名；

descriptor_index指向常量池中的一个CONSTANT_Utf8_info，该CONSTANT_Utf8_info描述了当前局部变量的描述符；

index描述了在该方法被执行时，当前局部变量在栈帧中局部变量表中的位置。

由此可知，方法中的每个局部变量都会对应一个local_variable_info。

#### Exceptions属性

首先需要说明，Exceptions属性不是存在于Code属性中的，它存在于method_info中的attributes中。和Code属性是平级的。这个属性描述的是方法声明的可能会抛出的异常，也就是方法定义后面的throws声明的异常列表，请不要和上面提到的异常处理器混淆。异常处理器描述了方法的字节码如何处理异常，而Exceptions属性描述方法可能会抛出哪些以异常。下面讲解Exceptions属性的结构（左下角为Exceptions属性）：

<a href="/img/post/jvm/42.png" target="_blank"><img src="/img/post/jvm/42.png" /></a>

下面讲解Exceptions属性中的信息。

attribute_name_index和attribute_length就不多说了， 和其他属性是一样的。 

number_of_exceptions是该方法要抛出的异常的个数。 

exceptions_index_table可以看做一个数组，这个数组中的每一项占两个字节，这两个字节是对常量池的索引，它指向一个常量池中的CONSTANT_Class_info。这个CONSTANT_Class_info描述了一个被抛出的异常的类型。

### 总结

到此为止，和方法相关的属性就介绍完了。这篇博客讲解的内容相对比较复杂。下面以一个实例进行验证，实例代码：

```java
package com.jg.zhang;
 
public class Test {
 
	public void test() throws Exception{
		
		int localVar = 0;
		
		try{
			
			Class.forName("com.jg.zhang.Person");
			
		}catch(ClassNotFoundException e){
			
			throw e;
		}finally{
			System.out.println(localVar);
		}
		
	}
}
```

反编译后的test方法部分（省略了常量池等信息）：

```java
  public void test() throws java.lang.Exception;
    flags: ACC_PUBLIC
    Exceptions:
      throws java.lang.Exception
    Code:
      stack=2, locals=4, args_size=1
         0: iconst_0
         1: istore_1
         2: ldc           #18                 // String com.jg.zhang.Person
         4: invokestatic  #20                 // Method java/lang/Class.forName:(Ljava/lang/String;)Ljava/lang/Class;
         7: pop
         8: goto          24
        11: astore_2
        12: aload_2
        13: athrow
        14: astore_3
        15: getstatic     #26                 // Field java/lang/System.out:Ljava/io/PrintStream;
        18: iload_1
        19: invokevirtual #32                 // Method java/io/PrintStream.println:(I)V
        22: aload_3
        23: athrow
        24: getstatic     #26                 // Field java/lang/System.out:Ljava/io/PrintStream;
        27: iload_1
        28: invokevirtual #32                 // Method java/io/PrintStream.println:(I)V
        31: return
      Exception table:
         from    to  target type
             2     8    11   Class java/lang/ClassNotFoundException
             2    14    14   any
      LineNumberTable:
        line 7: 0
        line 11: 2
        line 13: 8
        line 15: 12
        line 16: 14
        line 17: 15
        line 18: 22
        line 17: 24
        line 20: 31
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
               0      32     0  this   Lcom/jg/zhang/Test;
               2      30     1 localVar   I
              12       2     2     e   Ljava/lang/ClassNotFoundException;
```

结合上面的讲解和图解，再分析反编译的结果，就一目了然了：所有的结果是一个method_info，method_info开始处是访问标志信息。然后是method_info的 Exceptions属性，Exceptions属性属性下面是Code属性，Code属性中又包括字节码，异常处理器，LineNumberTable属性和LocalVariableTable属性。 

由于这篇博客讲解的内容大多和方法有关，所以会直接或间接的和method_info有联系，最后给出一张全局图，这样的话，读者就比较明确，一个完整的方法，是如何在class文件中描述的，由于考虑到复杂性，这些属性或其他数据项中，对常量池的引用均未画出：

<a href="/img/post/jvm/43.png" target="_blank"><img src="/img/post/jvm/43.png" /></a>


















































