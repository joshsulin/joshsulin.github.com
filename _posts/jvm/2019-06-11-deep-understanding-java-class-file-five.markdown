---
layout: post
title:  "深入理解Java Class文件格式(五)"
date:   2019-06-11 20:19:20 +0800
categories: Jvm
tags: Jvm
---

### 前情回顾

本专栏的前几篇博文， 对class文件中的常量池进行了详细的解释。 前文讲解了常量池中的7种数据项， 它们分别是：

CONSTANT_Utf8_info

CONSTANT_NameAndType_info

CONSTANT_Integer_info

CONSTANT_Float_info

CONSTANT_Long_info

CONSTANT_Double_info

CONSTANT_String_info

关于这七种数据项， 前面的文章已经讲得很详细了， 不了解的同学请先参阅前面的博文.

常量池中各数据项类型详解（续）

###（8） CONSTANT_Class_info

常量池中的一个CONSTANT_Class_info， 可以看做是CONSTANT_Class数据类型的一个实例。 他是对类或者接口的符号引用。 它描述的可以是当前类型的信息， 也可以描述对当前类的引用， 还可以描述对其他类的引用。 也就是说， 如果访问了一个类字段， 或者调用了一个类的方法， 对这些字段或方法的符号引用， 必须包含它们所在的类型的信息， CONSTANT_Class_info就是对字段或方法符号引用中类型信息的描述。 

CONSTANT_Class_info的第一个字节是tag， 值为7， 也就是说， 当虚拟机访问到一个常量池中的数据项， 如果发现它的tag值为7， 就可以判断这是一个CONSTANT_Class_info 。 tag下面的两个字节是一个叫做name_index的索引值， 它指向一个CONSTANT_Utf8_info， 这个CONSTANT_Utf8_info中存储了CONSTANT_Class_info要描述的类型的全限定名。 全限定名的概念在前面的博文 深入理解Java Class文件格式（二） 中将结果， 不熟悉的同学可以先阅读这篇文章。  

此外要说明的是， java中数组变量也是对象， 那么数组也就有相应的类型， 并且数组的类型也是使用CONSTANT_Class_info描述的， 并且数组类型和普通类型的描述有些区别。 普通类型的CONSTANT_Class_info中存储的是全限定名， 而数组类型对应的CONSTANT_Class_info中存储的是数组类型相对应的描述符字符串。 举例说明：

与Object类型对应的CONSTANT_Class_info中存储的是： java/lang/Object 

与Object[]类型对应的CONSTANT_Class_info中存储的是： [Ljava/lang/Object; 


下面看CONSTANT_Class_info的存储布局：

<a href="/img/post/jvm/16.png" target="_blank"><img src="/img/post/jvm/16.png" /></a>

例如， 如果在一个类中引用了System这个类， 那么就会在这个类的常量池中出现以下信息：

```java
Constant pool:

#16 = class      #17     // java/lang/System
#17 = Utf8       java/lang/System
```

###（9） CONSTANT_Fieldref_info

常量池中的一个CONSTANT_Fieldref_info， 可以看做是CONSTANT_Field数据类型的一个实例。 该数据项表示对一个字段的符号引用， 可以是对本类中的字段的符号引用， 也可以是对其他类中的字段的符号引用， 可以是对成员变量字段的符号引用， 也可以是对静态变量的符号引用, 其中ref三个字母就是reference的简写。 之前的文章中， “符号引用”这个名词出现了很多次， 可能有的同学一直不是很明白， 等介绍完CONSTANT_Fieldref_info， 就可以很清晰的了解什么是符号引用。 下面分析CONSTANT_Fieldref_info中的内容都存放了什么信息。 

和其他类型的常量池数据项一样， 它的第一个字节也必然是tag， 它的tag值为9 。 也就是说， 当虚拟机访问到一个常量池中的一项数据， 如果发现这个数据的tag值为9， 就可以确定这个被访问的数据项是一个CONSTANT_Fieldref_info， 并且知道这个数据项表示对一个字段的符号引用。 

tag值下面的两个字节是一个叫做class_index的索引值， 它指向一个CONSTANT_Class_info数据项， 这个数据项表示被引用的字段所在的类型， 包括接口。 所以说， CONSTANT_Class_info可以作为字段符号引用的一部分。 

class_index以下的两个字节是一个叫做name_and_type_index的索引， 它指向一个CONSTANT_NameAndType_info， 这个CONSTANT_NameAndType_info前面的博客中已经解释过了， 不明白的朋友可以先看前面的博客：深入理解Java Class文件格式（三） 。 这个CONSTANT_NameAndType_info描述的是被引用的字段的名称和描述符。 我们在前面的博客中也提到过， CONSTANT_NameAndType_info可以作为字段符号引用的一部分。

到此， 我们可以说， CONSTANT_Fieldref_info就是对一个字段的符号引用， 这个符号引用包括两部分， 一部分是该字段所在的类， 另一部分是该字段的字段名和描述符。 这就是所谓的 “对字段的符号引用” 。

下面结合实际代码来说明， 代码如下：

```java
package com.jg.zhang;
 
public class TestInt {
	int a = 10;
	void print(){
		System.out.println(a);
	}
}
```

在print方法中， 引用了本类中的字段a。 代码很简单， 我们一眼就可以看到print方法中是如何引用本类中定义的字段a的。 那么在class文件中， 对字段a的引用是如何描述的呢？ 下面我们将这段代码使用javap反编译， 给出简化后的反编译结果：

```java
Constant pool:
   #1 = Class              #2             //  com/jg/zhang/TestInt
   #2 = Utf8               com/jg/zhang/TestInt

   ......

   #5 = Utf8               a
   #6 = Utf8               I

   ......

  #12 = Fieldref           #1.#13         //  com/jg/zhang/TestInt.a:I
  #13 = NameAndType        #5:#6          //  a:I

  ......

{

  void print();
    flags:
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #19                 // Field java/lang/System.out:Ljava/io/PrintStream;
         3: aload_0
         4: getfield      #12                 // Field a:I
         7: invokevirtual #25                 // Method java/io/PrintStream.println:(I)V
        10: return
}
```

可以看到， print方法的位置为4的字节码指令getfield引用了索引为12的常量池数据项， 常量池中索引为12的数据项是一个CONSTANT_Fieldref_info， 这个CONSTANT_Fieldref_info又引用了索引为1和13的两个数据项， 索引为1的数据项是一个CONSTANT_Class_info， 这个CONSTANT_Class_info数据项又引用了索引为2的数据项， 索引为2的数据项是一个CONSTANT_Utf8_info , 他存储了字段a所在的类的全限定名com/jg/zhang/TestInt 。 而CONSTANT_Fieldref_info所引用的索引为13的数据项是一个CONSTANT_NameAndType_info， 它又引用了两个数据项， 分别为第5项和第6项， 这是两个CONSTANT_Utf8_info ， 分别存储了字段a的字段名a， 和字段a的描述符I 。 

下面给出内存布局图， 这个图中涉及的东西有点多， 因为CONSTANT_Fieldref_info引用了CONSTANT_Class_info和CONSTANT_NameAndType_info， CONSTANT_Class_info又引用了一个CONSTANT_Utf8_info ， 而CONSTANT_NameAndType_info又引用了两个CONSTANT_Utf8_info 。 

<a href="/img/post/jvm/17.png" target="_blank"><img src="/img/post/jvm/17.png" /></a>

###（10） CONSTANT_Methodref_info

常量池中的一个CONSTANT_Methodref_info， 可以看做是CONSTANT_Methodref数据类型的一个实例。 该数据项表示对一个类中方法的符号引用， 可以是对本类中的方法的符号引用， 也可以是对其他类中的方法的符号引用， 可以是对成员方法字段的符号引用， 也可以是对静态方法的符号引用,但是不会是对接口中的方法的符号引用。 其中ref三个字母就是reference的简写。 在上一小节中介绍了CONSTANT_Fieldref_info， 它是对字段的符号引用， 本节中介绍的CONSTANT_Methodref_info和CONSTANT_Fieldref_info很相似。既然是符号“引用”， 那么只有在原文件中调用了一个方法， 常量池中才有和这个被调用方法的相对应的符号引用， 即存在一个CONSTANT_Methodref_info。 如果只是在类中定义了一个方法， 但是没调用它， 则不会在常量池中出现和这个方法对应的CONSTANT_Methodref_info 。 

和其他类型的常量池数据项一样， 它的第一个字节也必然是tag， 它的tag值为10 。 也就是说， 当虚拟机访问到一个常量池中的一项数据， 如果发现这个数据的tag值为10， 就可以确定这个被访问的数据项是一个CONSTANT_Methodref_info， 并且知道这个数据项表示对一个方法的符号引用。 

tag值下面的两个字节是一个叫做class_index的索引值， 它指向一个CONSTANT_Class_info数据项， 这个数据项表示被引用的方法所在的类型。 所以说， CONSTANT_Class_info可以作为方法符号引用的一部分。 

class_index以下的两个字节是一个叫做name_and_type_index的索引， 它指向一个CONSTANT_NameAndType_info， 这个CONSTANT_NameAndType_info前面的博客中已经解释过了， 不明白的朋友可以先看前面的博客：深入理解Java Class文件格式（三） 。 这个CONSTANT_NameAndType_info描述的是被引用的方法的名称和描述符。 我们在前面的博客中也提到过， CONSTANT_NameAndType_info可以作为方法符号引用的一部分。

到此， 我们可以知道， CONSTANT_Methodref_info就是对一个字段的符号引用， 这个符号引用包括两部分， 一部分是该方法所在的类， 另一部分是该方法的方法名和描述符。 这就是所谓的 “对方法的符号引用” 。

下面结合实际代码来说明， 代码如下：

```java
package com.jg.zhang;
 
public class Programer {
 
	Computer computer;
	
	public Programer(Computer computer){
		this.computer = computer;
	}
	
	public void doWork(){
		computer.calculate();
	}
}
```

```java
package com.jg.zhang;
 
public class Computer {
 
	public void calculate() {
		System.out.println("working...");
		
	}
}
```

上面的代码包括两个类， 其中Programer类引用了Computer类， 在Programer类的doWork方法中引用（调用）了Computer类的calculate方法。源码中对一个方法的描述形式我们再熟悉不过了， 现在我们就反编译Programer， 看看Programer中对Computer的doWork方法的引用， 在class文件中是如何描述的。 

下面给出Programer的反编译结果， 其中省去了一些不相关的信息：

```java
Constant pool:
.........
 
 
  #12 = Utf8               ()V
 
 
  #20 = Methodref          #21.#23        //  com/jg/zhang/Computer.calculate:()V
  #21 = Class              #22            //  com/jg/zhang/Computer
  #22 = Utf8               com/jg/zhang/Computer
  #23 = NameAndType        #24:#12        //  calculate:()V
  #24 = Utf8               calculate
 
{
 
  com.jg.zhang.Computer computer;     
    flags:
 
.........
 
  public void doWork();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #13                 // Field computer:Lcom/jg/zhang/Computer;
         4: invokevirtual #20                 // Method com/jg/zhang/Computer.calculate:()V
         7: return
}
```

可以看到， doWork方法的位置为4的字节码指令invokevirtual引用了索引为20的常量池数据项， 常量池中索引为20的数据项是一个CONSTANT_Methodref_info， 这个CONSTANT_Methodref_info又引用了索引为21和23的两个数据项， 索引为21的数据项是一个CONSTANT_Class_info， 这个CONSTANT_Class_info数据项又引用了索引为22的数据项， 索引为22的数据项是一个CONSTANT_Utf8_info , 他存储了被引用的Computer类中的calculate方法所在的类的全限定名com/jg/zhang/Computer 。 而CONSTANT_Methodref_info所引用的索引为23的数据项是一个CONSTANT_NameAndType_info， 它又引用了两个数据项， 分别为第24项和第12项， 这是两个CONSTANT_Utf8_info ， 分别存储了被引用的方法calculate的方法名calculate， 和该方法的描述符()V 。 

下面给出内存布局图， 这个图中涉及的东西同样有点多， 因为CONSTANT_Methodref_info引用了CONSTANT_Class_info和CONSTANT_NameAndType_info， CONSTANT_Class_info又引用了一个CONSTANT_Utf8_info ， 而CONSTANT_NameAndType_info又引用了两个CONSTANT_Utf8_info 。 

<a href="/img/post/jvm/18.png" target="_blank"><img src="/img/post/jvm/18.png" /></a>

###（11） CONSTANT_InterfaceMethodref_info

常量池中的一个CONSTANT_InterfaceMethodref_info， 可以看做是CONSTANT_InterfaceMethodref数据类型的一个实例。 该数据项表示对一个接口方法的符号引用， 不能是对类中的方法的符号引用。 其中ref三个字母就是reference的简写。 在上一小节中介绍了CONSTANT_Methodref_info， 它是对类中的方法的符号引用， 本节中介绍的CONSTANT_InterfaceMethodref和CONSTANT_Methodref_info很相似。既然是符号“引用”， 那么只有在原文件中调用了一个接口中的方法， 常量池中才有和这个被调用方法的相对应的符号引用， 即存在一个CONSTANT_InterfaceMethodref。 如果只是在接口中定义了一个方法， 但是没调用它， 则不会在常量池中出现和这个方法对应的CONSTANT_InterfaceMethodref 。 

和其他类型的常量池数据项一样， 它的第一个字节也必然是tag， 它的tag值为11 。 也就是说， 当虚拟机访问到一个常量池中的一项数据， 如果发现这个数据的tag值为11， 就可以确定这个被访问的数据项是一个CONSTANT_InterfaceMethodref， 并且知道这个数据项表示对一个接口中的方法的符号引用。 

tag值下面的两个字节是一个叫做class_index的索引值， 它指向一个CONSTANT_Class_info数据项， 这个数据项表示被引用的方法所在的接口。 所以说， CONSTANT_Class_info可以作为方法符号引用的一部分。 

class_index以下的两个字节是一个叫做name_and_type_index的索引， 它指向一个CONSTANT_NameAndType_info， 这个CONSTANT_NameAndType_info前面的博客中已经解释过了， 不明白的朋友可以先看前面的博客：深入理解Java Class文件格式（三） 。 这个CONSTANT_NameAndType_info描述的是被引用的方法的名称和描述符。 我们在前面的博客中也提到过， CONSTANT_NameAndType_info可以作为方法符号引用的一部分。

到此， 我们可以知道， CONSTANT_InterfaceMethodref就是对一个接口中的方法的符号引用， 这个符号引用包括两部分， 一部分是该方法所在的接口， 另一部分是该方法的方法名和描述符。 这就是所谓的 “对接口中的方法的符号引用” 。

下面结合实际代码来说明， 代码如下：

```java
package com.jg.zhang;
 
public class Plane {
 
	IFlyable flyable;
	
	void flyToSky(){
		flyable.fly();
	}
}
```

```java
package com.jg.zhang;
 
public interface IFlyable {
 
	void fly();
}
```

在上面的代码中， 定义可一个类Plane， 在这个类中有一个IFlyable接口类型的字段flyable， 然后在Plane的flyToSky方法中调用了IFlyable中的fly方法。 这就是源代码中对一个接口中的方法的引用方式， 下面我们反编译Plane， 看看在class文件层面， 对一个接口中的方法的引用是如何描述的。

下面给出反编译结果， 为了简洁期间， 省略了一些不相关的内容：

```java
Constant pool:
.........

  #8 = Utf8               ()V

  #19 = InterfaceMethodref #20.#22        //  com/jg/zhang/IFlyable.fly:()V
  #20 = Class              #21            //  com/jg/zhang/IFlyable
  #21 = Utf8               com/jg/zhang/IFlyable
  #22 = NameAndType        #23:#8         //  fly:()V
  #23 = Utf8               fly

{

.........

  com.jg.zhang.IFlyable flyable;
    flags:

.........

  void flyToSky();
    flags:
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #17                 // Field flyable:Lcom/jg/zhang/IFlyable;
         4: invokeinterface #19,  1           // InterfaceMethod com/jg/zhang/IFlyable.fly:()V
         9: return

}
```

可以看到， flyToSky方法的位置为4的字节码指令invokeinterface引用了索引为19的常量池数据项， 常量池中索引为19的数据项是一个CONSTANT_InterfaceMethodref_info， 这个CONSTANT_InterfaceMethodref_info又引用了索引为20和22的两个数据项， 索引为20的数据项是一个CONSTANT_Class_info， 这个CONSTANT_Class_info数据项又引用了索引为21的数据项， 索引为21的数据项是一个CONSTANT_Utf8_info , 他存储了被引用的方法fly所在的接口的全限定名com/jg/zhang/IFlyable 。 而CONSTANT_InterfaceMethodref_info所引用的索引为22的数据项是一个CONSTANT_NameAndType_info， 它又引用了两个数据项， 分别为第23项和第8项， 这是两个CONSTANT_Utf8_info ， 分别存储了被引用的方法fly的方法名fly， 和该方法的描述符()V 。 

下面给出内存布局图， 这个图中涉及的东西同样有点多， 因为CONSTANT_InterfaceMethodref_info引用了CONSTANT_Class_info和CONSTANT_NameAndType_info， CONSTANT_Class_info又引用了一个CONSTANT_Utf8_info ， 而CONSTANT_NameAndType_info又引用了两个CONSTANT_Utf8_info 。

<a href="/img/post/jvm/19.png" target="_blank"><img src="/img/post/jvm/19.png" /></a>

### 总结

到此为止， class文件中的常量池部分就已经讲解完了。 进行一下总结。对于深入理解Java和JVM ， 理解class文件的格式至关重要， 而在class文件中， 常量池是一项非常重要的信息。 常量池中有11种数据项， 这个11种数据项存储了各种信息， 包括常量字符串， 类的信息， 方法的符号引用， 字段的符号引用等等。 常量池中的数据项通过索引来访问， 访问形式类似于数组。 常量池中的各个数据项之前会通过索引相互引用， class文件的其他地方也会引用常量池中的数据项 ， 如方法的字节码指令。 

在下面的文章中， 会继续介绍class文件中， 位于常量池以下的其他信息。 这些信息包括：对本类的描述， 对父类的描述， 对实现的接口的描述， 本类中声明的字段的描述， 本类汇总定义的方法的描述，还有各种属性。 
