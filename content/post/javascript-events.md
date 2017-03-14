+++
date = "2017-03-03T10:34:39+08:00"
title = "javascript events"
author = "wubing"
draft = false
categories = ["frontend"]
tags = ["javascript"]

+++
# JavaScript事件机制
事件是实现页面交互的常用手段，在浏览器环境下的事件主要由DOM、BOM事件类型组成。
## 一、什么是事件
事件就是文档、浏览器窗口中发生的一些特定的交互瞬间，一般是由用户主动触发，在事件被触发之后，可以根据是否预先绑定了事件处理程序来决定如何对这个事件的发生做出反应。
<!--more-->
## 二、事件流
事件流是指页面中的元素、节点接收某一个被触发的事件的顺序。分为事件冒泡、事件捕获和DOM事件流。
IE和Netscape的浏览器开发团队提出了两种截然不同的，甚至是完全相反的事件流概念。IE的事件流是事件冒泡，而Netscape的事件流是事件捕获。

  - 事件冒泡：事件发生后，由目标元素首先接收，然后按照DOM结构，逐级向上传播，直至到达最顶层的根节点document
  - 事件捕获：事件发生后，由根节点document先接收，然后按照DOM结构，逐级向下传播，直到事件的目标元素
  - DOM事件流：包括三个阶段：当一个事件发生，首先进入事件捕获阶段，按照事件捕获流进行传播；然后进入目标阶段，目标元素接收到事件；最后进入事件冒泡阶段，按照事件冒泡流传播。![image](http://7xi480.com1.z0.glb.clouddn.com/121ba0d37798d6d23227d2dd49d0e58c_articlex.png)
## 三、基础事件操作
### 1、事件监听
浏览器会根据某些操作触发对应事件，比如我们打开一个页面，浏览器加载完成之后就会触发load事件。当我们监听load事件，显示欢迎信息，那么浏览器在加载完成后就会显示欢迎信息。

    - html内联属性：HTML元素里面直接填写与事件有关属性，属性值为JavaScript代码。
    ```
    <button onclick="alert('you clicked this btn')">click me！</button>
    ```

    - DOM属性绑定：使用DOM元素的onXXX属性设置，简单易懂，兼容性好，但是如果在后面的代码中再次为ele绑定一个回调函数，会覆盖掉之前回调函数的内容。
    ```
           <button id="btn">click me!</button>
           <script>
               var ele = document.getElementById('btn');
               ele.onclick = function (event) {
                   alert('you clicked again');
               }
           </script>
    ```

    - 使用事件监听函数：标准的事件监听函数如下：
    ```
    element.addEventListener(event-name,callback,use-capture)
    ```

    在element这个对象上面添加一个事件监听器，当监听到有事件发生时，调用这个回调函数，use-capture表示该事件监听是在‘捕获’阶段监听（设置为true）还是在‘冒泡’阶段监听（设置为false）
    ```
    <button>click me!</button>
    <script>
        var ele = document.getElementByTagName('button');
        ele[0].addEventListener('click', function () {
            alert('the third time clicked');
        }, false);
    </script>
    ```
### 2、移除事件监听
当我们为某个元素绑定了一个事件，每次触发这个事件的时候，都会执行事件绑定的回调函数。如果我们想解除绑定，需要使用 removeEventListener 方法：
```
element.removeEventListener(event-name,callback,use-capture)
```
需要注意的是，绑定事件时的回调函数不能是匿名函数，必须是一个声明的函数，因为解除事件绑定时需要传递这个回调函数的引用，才可以断开绑定。
```
<button id="btn">click me</button>
<script>
    var elbtn = document.getElementById('btn');
    var fun = function () {
        alert(this button is only can click once);
        elbtn.removeEventListener('click', fun, false);
    };
    elbtn.addEventListener('click', fun, false);
</script>
```
# window事件
## 常用的window事件
例子：https://jsfiddle.net/uujt1o1p/
- ### onload加载事件和onunload卸载事件
当页面完全加载之后触发加载事件（onload）
```
  window.onload = function () {
      alert("here is after  onload");
  }
  alert('onloading...');
```
当页面完全卸载之后触发卸载事件(unload)，在页面从服务器获取到需要加载的新的页面后调用，不能阻止页面的刷新和关闭。
```
  //firefox不支持，IE和360生效，chrome系统自定义
  var isSave = false;
  function save() {
     alert('this is save function running.');
      isSave = true;
  }
  window.onbeforeunload = function () {
      console.log('this is onbeforeunload');
      if (!isSave) {
          return '当前数据还没保存，关闭或刷新窗口会自动保存，是否继续？';
      } else {
          return '';
      }
  };
  window.onunload = function () {
      if (!isSave) {
          save();
      }
  };
```
onbeforeunload事件：在页面去服务器读取新的页面前调用，可以阻止页面的关闭和刷新，还有unload事件的执行
so:

   - 页面加载时只执行onload
   - 页面关闭时先执行onbeforeunload，最后onunload
   - 页面刷新时先执行onbeforeunload，然后onunload，最后onload。
- ### onscroll滚动事件
当滚动的时候触发滚动事件
```
window.onscroll = function() {
          console.log("滚动");

          var h = document.documentElement.scrollTop || document.body.scrollTop;
          console.log(h);
};
```
 document.documentElement.scrollTop 和document.body.scrollTop：是用来设置滚动tiao的高度。
 document.documentElement.scrollTop：IE，firefox；
 document.body.scrollTop: chrome,360

    ```
    // 返回页面顶部
    function fun() {
        document.documentElement.scrollTop = 0;
        document.body.scrollTop = 0;
    }
    ```
- ### onresize窗口变化事件
改变窗口尺寸时触发窗口变化事件
```
window.onresize = function() {
          console.log("窗口变化");

          var w = document.documentElement.clientWidth || document.body.clientWidth || window.innerWidth;

          var h = document.documentElement.clientHeight || document.body.clientHeight || window.innerHeight;

          console.log(w, h);
};
```

# 键盘事件
js中的键盘事件只有三种：keydown、keypress、keyup。触发顺序为：keydown ——> keypress ——> keyup。
当按下一个键不放开时，一般会重复触发keydown+keypress，直到放开后触发一个keyup事件。

- keydown()：在键盘按下时触发

- keypress()：在敲击按键时ch，即按下并抬起同一个按键
- keyup()：在按键释放时触发
## 获取键盘上的ascII码
栗子：https://jsfiddle.net/fz87pk7t/1/

```
// 获取键码
$(document).keydown(function (event) {
    alert(event.keyCode + 'keydown');
});

$(document).keyup(function (event) {
    alert(event.keyCode + 'keyup');
});

$(document).keypress(function (event) {
    alert(event.keyCode + 'keypress');
});
```
字母和数字键的键码值(keyCode)

按键 | 键码 | 按键 | 键码 | 按键 | 键码 | 按键 | 键码
---|---|---|---|---|---|---|---
A | 65 | J | 74 | S | 83 | 1 | 49
B | 66 | K | 75 | T | 84 | 1 | 50
C | 67 | L | 76 | U | 85 | 1 | 51
D | 68 | M | 77 | V | 86 | 1 | 52
E | 69 | N | 78 | W | 87 | 1 | 53
F | 70 | O | 79 | X | 88 | 1 | 54
G | 71 | P | 80 | Y | 89 | 1 | 55
H | 72 | Q | 81 | Z | 90 | 1 | 56
I | 73 | R | 82 | 0 | 48 | 1 | 57

数字键盘上的键的键码值(keyCode)

按键 | 健码 | 按键 | 健码
--- | --- | --- | ---
0 | 96 | 8 | 104
1 | 97 | 9 | 105
2 | 98 | * | 106
3 | 99 | + | 107
4 | 100 | Enter | 108
5 | 101 | - | 109
6 | 102 | . | 110
7 | 103 | / | 111

功能键键码值(keyCode)

按键 | 健码 | 按键 | 健码
---|---|---|---
F1 | 112 | F7 | 118
F2 | 113 | F8 | 119
F3 | 114 | F9 | 120
F4 | 115 | F10 | 121
F5 | 116 | F11 | 122
F6 | 117 | F12 | 123

控制键键码值(keyCode)

按键 | 键码	| 按键 | 键码 | 按键 | 键码 | 按键 | 键码
---|---|---|---|---|---|---|---
BackSpace |	8 |	Esc | 27 | Right Arrow | 39 | -_ | 189
Tab	| 9	| Spacebar | 32 | Down Arrow | 40 | .> | 190
Clear | 12 | Page Up | 33 |	Insert | 45 | /? | 191
Enter |	13 | Page Down | 34 | Delete | 46 |	`~ | 192
Shift |	16 | End | 35 |	Num Lock | 144 | [{ | 219
Control	| 17 | Home | 36 | ;: |	186 | \ \| |220
Alt | 18 | Left Arrow | 37 | =+ | 187 |	]} | 221
Cape Lock |	20 | Up Arrow |	38 | ,< | 188 |	'"	| 222

## 三种键盘事件的区别
- keydown 和 keyup 是比较底层的事件，分别对应一个键的按下与放开，键盘上任何一个键被点按，都会触发这两个事件。
- keypress 更接近用户，只有可打印的字符和控制字符（如换行）被键入才触发。
- keydown 和 keyup 被触发后，都会产生一个虚拟键盘码，用来表示按下的是哪个键。键盘上的每个键只对应一个虚拟键盘码。keypress 被触发后产生的是实际键入的字符的 Ascll 编码，而没有对应的虚拟键盘码。这两种编码有时在数值上是相等的。

#### 注意：
- keypress：只针对一些可以打印出来的字符有效，而对于功能键如F1-F12、Backspace、Enter、Escape、PageUp、PageDown和箭头方向等，就不会产生keypress事件，但是可以产生keydown、keyup事件
- keypress的keyCode对字母大小写敏感，而keydown和keyup不敏感；
- keypress事件无法区分主键盘上的数字键和副键盘数字键，而keydown和keyup的keyCode可以区分。


# 鼠标事件MouseEvent
常见的鼠标事件：

- mousedown:鼠标的犍钮被按下
- mouseup:鼠标的犍钮被释放弹起
- click：单击，mousedown + mouseup => click
- dblclick:双击，连续两次单击
- contextmenu:弹出右键菜单
    - el.contextmenu(handle)或el.on('contextmenu',handle)
    - el.off('contextmenu'),移除事件
- mouseover:鼠标移到目标上方
- mouseout:鼠标移出目标的上方
- mousemove:鼠标在目标上移动
- hover():鼠标指针进入和离开元素事被执行，同时绑定mouseenter和mouseleave事件，el.hover(handlerIn,handlerOut)等同于el.mouseenter(handlerIn).mouseleave(handlerOut);
- MouseEvent.button:返回值为数值(在DOM2.0中，W3C对鼠标事件作了现范，鼠标事件被解析为MouseEvent（我们可以用e.constructor == MouseEvent来判断其是否为鼠标事件，是左键点击还是右键点击由它的一个叫button的属性判定。以下就是W3C的标准现范：)
    - 1：左键被按下
    - 2：中键被按下（没测出来中键）
    - 3：右键被按下

栗子：https://jsfiddle.net/9gjz3eux/1/

