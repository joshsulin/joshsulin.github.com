---
layout: post
title:  "javascript里的匿名函数和闭包"
date:   2019-06-10 20:19:20 +0800
categories: Nodejs
tags: Nodejs
---

匿名函数就是没有名字的函数, 闭包是可访问一个函数作用域里变量的函数.

### 一．匿名函数

```javascript
//普通函数(函数名box())
function box() {
		return 'Lee';
}
console.log(box());

//匿名函数, 会报错
function() {
		return 'Lee';
}

//通过表达式自我执行
(function() {
		alert('Lee');
})(); //()表示执行函数, 并且传参

//把匿名函数赋值给变量
var box = function() {
		return 'Lee';
};
console.log(box());  //调用方式和函数调用相似

//函数里的匿名函数
function box() {
		return function() {  // 函数里的匿名函数, 产生闭包
				return 'Lee';
		}
}
console.log(box()());  // 调用匿名函数
```

### 二：闭包

闭包的概念：闭包是指有权访问另一个函数作用域中的变量的函数，创建闭包的常见的方式，就是在 一个函数内部创建另一个函数，通过另一个函数访问这个函数的局部变量。

//通过闭包可以返回局部变量

```javascript
function a(){
		var name='sun';
    return function (){  //通过匿名函数返回 a()局部变量 
    		return name; 
		}
}    
alert(a()());            //通过a()()来直接调用匿名函数返回值

var b = a();
alert(b());              //另一种调用匿名函数返回值
```

使用闭包有一个优点，也是它的缺点：就是可以把局部变量驻留在内存中，可以避免使用全局变量。

(全局变量污染导致应用程序不可预测性，每个模块都可调用必将引来灾难， 所以推荐使用私有的，封装的局部变量)。

#### 闭包的经典案例

通过全局变量来累加

```javascript
var num=0;     //全局变量 
function a(){
  num++;               //模块级可以调用全局变量，进行累加
}
a();   //1
a();   //2           //执行函数，累加了 
alert(num);      //输出全局变量

<--------------------------------------------------------------->

function a(){
		var num=0;
    num++;
    return num;
}

alert(a());        //1
alert(a());        //1               //无法实现累加，因为局部变量又被初始化了
```

使用闭包进行累加

```javascript
function a(){
		var num=0;
    return function(){
    		num++;
    		return num;
		}
}

var b = a();      //获得函数 

alert(b());  //1     //调用匿名函数 
alert(b());  //2     //第二次调用匿名函数，实现累加
```

PS：由于闭包里作用域返回的局部变量资源不会被立刻销毁回收，所以可能会占用更多的内存。过度使用闭包会导致性能下降，建议在非常有必要的时候才使用闭包。
