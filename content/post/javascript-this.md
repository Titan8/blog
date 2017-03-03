+++
date = "2017-03-03T09:41:07+08:00"
title = "小谈javascript中的this"
tags = ["javascript"]
categories = ["frontend"]
draft = false

+++
# this的用法
在javascript中，因为函数的调用方式的不同，this可以是全局对象、当前对象、任意对象。
## 1. 全局中
在全局中（任何函数体外部），this指代全局对象，无论是否在严格模式下。
```
在浏览器中，全局对象为window对象
console.log(this); // window

this.a = 1;
console.log(window.a); // 1
```
## 2. 函数体中
在函数内部，函数的调用方式决定了this的值
### 2.1 直接调用
```
非严格模式下：
function f1() {
    return this;
}

f1(); // window

严格模式下：
function f2() {
    "use strict";
    return this;
}

f2(); // undefined
```
<!--more-->
### 2.2 对象方法中的this
当以对象的方法的方式调用函数时，函数内的this指调用该函数的对象。
```
var obj = {
    age: 25,
    fun: function() {
        return this.age;
    }
};

console.log(obj.fun()); // 25
```
this的绑定只受最靠近的成员引用的影响
```
var obj = {age: 24};
function fun1(){
    return this.age;
}

obj.fun = fun1;
console.log(obj.fun()); // 24

obj.b = {
    f: fun1,
    age: 18
};
console.log(obj.b.f()); // 18
```
(1). 原型链中的this
```
var obj = {
	fun: function() {
		return this.a + this.b;
	}
};
var p = new Object(obj);
p.a = 1;
p.b = 2;
console.log(p.fun()); // 3 对象p的fun属性继承自它的原型，但是因为fun是作为p的方法调用的，所以它的this指向了p
```
(2). getter与setter中的this
```
function f1() {
	return this.a*this.a + this.b * this.b;
}
var obj = {
	a: 1,
	b: 2,
	get f2() {
		return this.a+this.b;
	}
};
Object.defineProperty(obj, 'f1', {
	get: f1, enumerable:true,configurable:true
});

console.log(obj.f1, obj.f2); // 5   3  作为getter或setter函数都会绑定 this 到从设置属性或得到属性的那个对象。
```
### 2.3 构造函数中的this
当一个函数被作为一个构造函数来使用（使用new关键字），它的this与即将被创建的新对象绑定。
```
function f1() {
    this.a = 1;
}

var obj = new f1();
console.log(obj.a); // 1

function f2() {
    this.a = 2;
    return {a: 3};
}

obj = new f2();
console.log(obj.a); // 3  因为手动设置了返回对象，所以this.a=2虽然执行了，但是对外部没有任何影响
```
### 2.4 call和apply
每个函数都包含两个非继承而来的方法：call()和apply()。这两个方法的用途都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值。也就是说，直接调用函数，调用时指定执行环境是谁.
```
window.age = 1;
var obj = {age: 2};

function sayAge() {
    console.log(this.age);
}

sayAge.call(this); // 1
sayAge.call(window); // 1
sayAge.call(obj); // 2
```
apply方法接收两个参数，第一个参数是在函数中运行函数的作用域，另一个参数是数组。
call方法与apply相比较，第一个参数相同，用于指定函数运行的作用域，其余参数直接传递给函数。
```
function add(c, d) {
    return this.a + this.b + c + d;
}

var obj = {a: 1, b: 2};
add.call(obj, 3, 4); // 1+2+3+4 = 10
add.apply(obj, [3,4]); // 1+2+3+4 = 10

注意： 使用call和apply时，如果传递的this值不是yijavascript会默认使用ToObject将其转换为对象

function bar() {
    console.log(this);
}

bar.call(7); // number
bar.call('fun'); // string
```
### 2.5 bind方法
ECMAScript 5 引入了 Function.prototype.bind。调用f.bind(someObject)会创建一个与f具有相同函数体和作用域的函数，但是在这个新函数中，this将永久地被绑定到了bind的第一个参数，无论这个函数是如何被调用的。
```
function f() {
	return this.a;
}

var g=f.bind({a: 'jing'});
console.log(g()); // jing

var obj = {
    a: liu,
    f: f,
    g: g
}

console.log(obj.f()); // liu
console.log(obj.g()); // jing
```
### 2.6 DOM事件处理函数中的this
当函数被用作事件处理函数时，它的this指向触发事件的元素,例子：https://jsfiddle.net/3urqmtb0/
```
<html>
   <body>
      <button>btn1</button>
      <button>btn2</button>
      <button>btn3</button>
   </body>
</html>
 // 点击当前元素，元素背景色变为粉色
<script>
    function pinkify(event) {
    	console.log(this === event.currentTarget);
        console.log(this);
        this.style.backgroundColor = 'pink';
    }
    var elements = document.getElementsByTagName('button');

    for(var i=0; i<elements.length; i++) {
    	elements[i].addEventListener('click', pinkify, false);
    }
</script>
```
### 2.7 内联事件处理函数中的this
当代码被内联处理函数调用时，它的this指向监听器所在的DOM元素，例子：https://jsfiddle.net/ak7sscuL/
```
<button onclick="alert(this.tagName.toLowerCase());">
  Show this
</button>
// alert button

<button onclick="alert((function(){return this})());">
  Show inner this
</button>
// alert [Object Window]  没有设置内部函数的 this，所以它指向 global/window 对象
```


