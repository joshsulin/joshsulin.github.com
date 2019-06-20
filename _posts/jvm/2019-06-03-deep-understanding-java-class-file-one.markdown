---
layout: post
title:  "深入理解Java Class文件格式(一)"
date:   2019-06-03 20:19:20 +0800
categories: Jvm
tags: Jvm
---

### Class文件在Java体系结构中的位置和作用

在上一篇博客中， 大致讲解了Java虚拟机的体系结构和执行原理。 本篇博客主要讲解能够被JVM识别， 加载并执行的class文件的格式。

对于理解JVM和深入理解Java语言， 学习并了解class文件的格式都是必须要掌握的功课。原因很简单， JVM不会理解我们写的Java源文件， 我们必须把Java源文件编译成class文件， 才能被JVM识别， 对于JVM而言， class文件相当于一个接口， 理解了这个接口， 能帮助我们更好的理解JVM的行为；另一方面， class文件以另一种方式重新描述了我们在源文件中要表达的意思， 理解class文件如何重新描述我们编写的源文件， 对于深入理解Java语言和语法都是很有帮助的。 另外， 不管是什么语言， 只要能编译成class文件， 都能被JVM识别并执行， 所以class文件不仅是跨平台的基础， 也是JVM跨语言的基础， 理解了class文件格式， 对于我们学习基于JVM的其他语言会有很大帮助。 

总之， 在整个Java技术体系结构中， class文件处于中间的位置， 对于理解整个体系有着承上启下的作用。 如图所示：

<a href="/img/post/jvm/4.png" target="_blank"><img src="/img/post/jvm/4.png" /></a>

### Class文件格式概述

class文件是一种8位字节的二进制流文件， 各个数据项按顺序紧密的从前向后排列， 相邻的项之间没有间隙， 这样可以使得class文件非常紧凑， 体积轻巧， 可以被JVM快速的加载至内存， 并且占据较少的内存空间。我们的Java源文件， 在被编译之后， 每个类（或者接口）都单独占据一个class文件， 并且类中的所有信息都会在class文件中有相应的描述， 由于class文件很灵活， 它甚至比Java源文件有着更强的描述能力。

class文件中的信息是一项一项排列的， 每项数据都有它的固定长度， 有的占一个字节， 有的占两个字节， 还有的占四个字节或8个字节， 数据项的不同长度分别用u1, u2, u4, u8表示， 分别表示一种数据项在class文件中占据一个字节， 两个字节， 4个字节和8个字节。 可以把u1, u2, u3, u4看做class文件数据项的“类型” 。

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

下面对class文件中的每一项进行详细的解释。

### class文件中的魔数和版本号

（1） magic

在class文件开头的四个字节， 存放着class文件的魔数， 这个魔数是class文件的标志，他是一个固定的值： 0XCAFEBABE 。 也就是说他是判断一个文件是不是class格式的文件的标准， 如果开头四个字节不是0XCAFEBABE， 那么就说明它不是class文件， 不能被JVM识别。

（2）minor_version 和 major_version

紧接着魔数的四个字节是class文件的此版本号和主版本号。 随着Java的发展， class文件的格式也会做相应的变动。 版本号标志着class文件在什么时候， 加入或改变了哪些特性。 举例来说， 不同版本的javac编译器编译的class文件， 版本号可能不同， 而不同版本的JVM能识别的class文件的版本号也可能不同， 一般情况下， 高版本的JVM能识别低版本的javac编译器编译的class文件， 而低版本的JVM不能识别高版本的javac编译器编译的class文件。 如果使用低版本的JVM执行高版本的class文件， JVM会抛出java.lang.UnsupportedClassVersionError 。具体的版本号变迁这里不再讨论， 需要的读者自行查阅资料。 

### class文件中的常量池概述

在class文件中， 位于版本号后面的就是常量池相关的数据项。 常量池是class文件中的一项非常重要的数据。 常量池中存放了文字字符串， 常量值， 当前类的类名， 字段名， 方法名， 各个字段和方法的描述符， 对当前类的字段和方法的引用信息， 当前类中对其他类的引用信息等等。 常量池中几乎包含类中的所有信息的描述， class文件中的很多其他部分都是对常量池中的数据项的引用，比如后面要讲到的this_class, super_class, field_info, attribute_info等， 另外字节码指令中也存在对常量池的引用， 这个对常量池的引用当做字节码指令的一个操作数。  此外， 常量池中各个项也会相互引用。

class文件中的项constant_pool_count的值为1, 说明每个类都只有一个常量池。 常量池中的数据也是一项一项的， 没有间隙的依次排放。常量池中各个数据项通过索引来访问， 有点类似与数组， 只不过常量池中的第一项的索引为1, 而不为0, 如果class文件中的其他地方引用了索引为0的常量池项， 就说明它不引用任何常量池项。class文件中的每一种数据项都有自己的类型， 相同的道理，常量池中的每一种数据项也有自己的类型。 常量池中的数据项的类型如下表：

| 常量池中数据项类型 | 类型标志 | 类型描述 |
| --------           | :-----   | :----    |
| CONSTANT_Utf8      | 1        | UTF-8编码的Unicode字符串 |
| CONSTANT_Integer   | 3        | int类型字面值            |
| CONSTANT_Float     | 4        | float类型字面值          |
| CONSTANT_Long      | 5        | long类型字面值           |
| CONSTANT_Double    | 6        | double类型字面值         |
| CONSTANT_Class     | 7        | 对一个类或接口的符号引用 |
| CONSTANT_String    | 8        | String类型字面值 |
| CONSTANT_Fieldref  | 9        | 对一个字段的符号引用 |
| CONSTANT_Methodref | 10       | 对一个类中声明的方法的符号引用 |
| CONSTANT_InterfaceMethodref | 11       | 对一个接口中声明的方法的符号引用 |
| CONSTANT_NameAndType | 12       | 对一个字段或方法的部分符号引用 |

每个数据项叫做一个XXX_info项， 比如， 一个常量池中一个CONSTANT_Utf8类型的项， 就是一个CONSTANT_Utf8_info。除此之外， 每个info项中都有一个标志值（tag）， 这个标志值表明了这个常量池中的info项的类型是什么， 从上面的表格中可以看出， 一个CONSTANT_Utf8_info中的tag值为1， 而一个CONSTANT_Fieldref_info中的tag值为9 。

Java程序是动态链接的， 在动态链接的实现中， 常量池扮演者举足轻重的角色。 除了存放一些字面量之外， 常量池中还存放着以下几种符号引用：

（1） 类和接口的全限定名

（2） 字段的名称和描述符

（3） 方法的名称和描述符

在详细讲解常量池中的各个数据项之前， 我们有必要先了解一下class文件中的特殊字符串， 因为在常量池中， 特殊字符串大量的出现，这些特殊字符串就是上面说的全限定名和描述符。 要理解常量池中的各个数据项， 必须先了解这些特殊字符串。

对于class文件的讲解会在后续博文中继续， 欢迎关注。
