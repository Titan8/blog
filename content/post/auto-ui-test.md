+++
draft = false
date = "2017-01-22T17:03:51+08:00"
title = "RF中页面元素定位"
categories = ["test"]
tags = ["autotest"]
+++

# RF中页面元素定位
### 1.Robot Framework+selenium做UI自动化测试

### 2.页面元素定位
### 2.1 id和name定位
假如把一个元素看作一个人的话，id 和name可以看作一个人的身份证号和姓名。当然，这些属性值是否唯一要看前端工程师如何设计了。

百度搜索框和搜索按钮
```html
<input type="text" class="s_ipt" ==name="wd"== id="kw" maxlength="100" autocomplete="off">

<input type="submit" value="百度一下" ==id="su"== class="btn self-btn bg s_btn">

```

<!--more-->

根据上面的例子，百度输入框可以取id 或name进行定位。（前提是id和name的值在当页面上唯一）
```html
id = su
name = wd
```
在Robot framework 中写法如下：

方法 | 定位|文本
---|---|---
Input Text | id=kw | 地图| 
Input Text | name=wd |地图| 

百度按钮只id数据可以利用：id = su

方法 | 定位|文本
---|---|---
Click Button | id=su|
Click Button是按钮点击的关键字。
### 2.2 xpath定位
假如页面元素。既没有id也没有name
例如：cgntes的登录页面
```html
<input class="input ng-pristine ng-valid ng-empty ng-touched" autocomplete="off" ng-change="vm.loginFailed=false" ng-model="vm.user.username" placeholder="请输入用户名" autofocus="" ng-keyup="vm.keyUp($event)" tabindex="1" aria-invalid="false" style="">
```
 
1. Xpath的绝对路径：

```html
Xpath =/html/body/div/titan-login/div/div[1]/div[2]/input
```
我们可以从最外层开始找，html下面的body下面的div下面的第4个div下面的....input标签。通过一级一级的锁定就找到了想要的元素。
 
2. Xpath的相对路径：

绝对路径的用法往往是在我们迫不得已的时候才用的。大多时候用相对路径更简便。
3. 元素本身：
Xpath同样可以利用元素自身的属性：

```html
Xpath=/html/body/div/titan-login/div/div[1]/div[2]/input[@placeholder='请输入用户名']
Xpath = //*[@placeholder='请输入用户名]
```

//表示某个层级下，*表示某个标签名。@placeholder='请输入用户名'表示这个元素有个placeholder等于请输入用户名 。
元素本身，可以利用的属性就不只局限为于id和name，但要保证这些元素可以唯一的识别一个元素。
 
4.找上级：

当我们要找的元素没有id和name，但是它有父节点，那么同样可以定位到该元素。

元素的上级属性为：
```html
<form name="f" id="form" action="/s" class="fm" onsubmit="javascript:F.call('ps/sug','pssubmit')">
　　<span class="s_kw_wrap" class="bg s_ipt_wr quickdelete-wrap">
  <span class="soutu-btn">
　　　 <input  type="text" class="s_ipt" name="wd" id="kw" maxlength="100" autocomplete="off">
```
找父节点：

```html
xpath = //span[@class=’bg s_ipt_wr’]/input
```
如果父节点没有唯一的属性，可以找父父节点：
xpath = //form[@id=’form’]/span/input
这样一级一级找上去，直到html那么就是一个绝对路径了。
### 2.3 CSS定位
CSS的定位更灵活，因为他它用到的更多的匹配符和规格。

选择器| 示例|示例描述
---|---|---
.class | .intro|选择 class="intro" 的所有元素||
id|firstname|选择 id="firstname" 的所有元素
*|*|选择所有元素|
element|p|选择所有 <p> 元素|
element.element|div.p|选择所有 <div> 元素和所有 <p> 元素|
element>element|div>p|选择父元素为 <div> 元素的所有 <p> 元素|
element+element|div+p|选择紧接在 <div> 元素之后的所有 <p> 元素|
[attribute]|[target]|选择带有 target 属性所有元素|
[attribute=value]|[target=_blank]|选择 target="_blank" 的所有元素|
[attribute~=value]|[title~=flower]|选择 title 属性包含单词 "flower" 的所有元素|
[attribute=value]|[lang=en]|选择 lang 属性值以 "en" 开头的所有元素|

同样以百度输入框的代码，具体CSS定位如下：

id定位：
```html
css=#kw

class定位：

css=.s_ipt
```

其它属性：
```html
css=[name=wd]
css=[type=text]
css=[autocomplete=off]
```
父子定位：
```html
 css=span>input
 css=form>span>input
```
根据标签名定位：
```html
css=input
```

以Cgntest的登录按钮为例子Robot framework 中的写法：

方法 | 定位|文本
---|---|---
Input Text | css=.btn | \\\13|
Input Text | css=body > div > titan-login > div > div.form > div.btn|\\\13|

同样一个元素，根基CSS的不同规则，可能有几十上百种写法。CSS更灵活强大，但是相比Xpath 的学习成本为更高。但是CSS和Xpath两种定位方式是一定要学会一种，不然你的自动化工作更无法开展。
### 2.4 JS定位
当碰到使用上述方法仍无法定位时，可选择JS定位，该方法比较通用，能正常定位到所有元素。

方法 | 定位
---|---|
Execute JavaScript | $(".btn").click() |

