+++
draft = false
date = "2017-01-24T16:50:51+08:00"
title = "D3 入门教程"

categories = [
    "frontend"
]
tags = ["JavaScript"]

+++
# 1. 数据可视化
数据可视化，是关于数据视觉表现形式的科学技术研究。为了使复杂的数据和文字变得更容易理解，各种可视化工具因此诞生，其中D3 正是其中的佼佼者。
![image](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/06/14.png)
# 2. D3 简介
D3 的全称是（Data-Driven Documents），字面意思是一个被数据驱动的文档。听名字有点抽象，说简单一点，其实就是一个 JavaScript 的函数库，使用它主要是用来做数据可视化的。
# 3. 基本用法
## 3.1 选择元素
在 D3 中，用于选择元素的函数有两个，函数接收符合CSS选择器条件的字符串：
- d3.select()：是选择所有指定元素的第一个
- d3.selectAll()：是选择指定元素的全部

这两个函数返回的结果称为选择集。

<!--more-->

## 3.2 绑定数据
D3 有一个很独特的功能：能将数据绑定到 DOM 上，绑定之后，当需要依靠这个数据才操作元素的时候，会很方便。

D3 中是通过以下两个函数来绑定数据的：

- datum()：绑定一个数据到选择集上
- data()：绑定一个数组到选择集上，数组的各项值分别与选择集的各元素绑定

相对而言，data() 比较常用。
## 3.3 插入元素
插入元素涉及的函数有两个：

- append()：在选择集末尾插入元素
- insert()：在选择集前面插入元素

## 3.4 删除元素
对于选择的元素，使用remove()函数

## 3.5 比例尺
将某一区域的值映射到另一区域，其大小关系不变。

比例尺，很像数学中的函数。例如，对于一个一元二次函数，有 x 和 y 两个未知数，当 x 的值确定时，y 的值也就确定了。

在数学中，x 的范围被称为定义域，y 的范围被称为值域。

D3 中的比例尺，也有定义域和值域，分别被称为 domain 和 range。开发者需要指定 domain 和 range 的范围，如此即可得到一个计算关系。

D3 提供了多种比例尺，常用的有以下两种：
- d3.scale.linear()：返回一个线性的比例尺，能将一个连续的区间，映射到另一区间。
- d3.scale.ordinal()：返回一个序数比例尺，即定义域和值域是离散的。

## 3.6 坐标轴
坐标轴，是可视化图表中经常出现的一种图形，由一些列线段和刻度组成。在D3 中使用d3.svg.axis()，能够在 SVG 中生成组成坐标轴的元素。

# 4. 实现一个简单的柱形图
一个完整的柱形图包含三部分：矩形、文字、坐标轴。基于D3 的基本用法，制作一个简单的柱形图，内容包括：选择集、数据绑定、比例尺、坐标轴等内容。
## 4.1 添加 SVG 画布

```
//画布大小
var width = 400;
var height = 400;

//在 body 里添加一个 SVG 画布   
var svg = d3.select("body")
    .append("svg")
    .attr("width", width)
    .attr("height", height);

//画布周边的空白
 var padding = {left:30, right:30, top:20, bottom:20};
```
上面定义了一个 padding，是为了给 SVG 的周边留一个空白，最好不要将图形绘制到边界上。

## 4.2 定义数据和比例尺

```
//定义一个数组
var dataset = [10, 20, 30, 40, 33, 24, 12, 5];

//x轴的比例尺
var xScale = d3.scale.ordinal()
    .domain(d3.range(dataset.length))
    .rangeRoundBands([0, width - padding.left - padding.right]);

//y轴的比例尺
var yScale = d3.scale.linear()
    .domain([0,d3.max(dataset)])
    .range([height - padding.top - padding.bottom, 0]);
```
x 轴使用序数比例尺，y 轴使用线性比例尺。要注意两个比例尺值域的范围。

## 4.3 定义坐标轴

```
//定义x轴
var xAxis = d3.svg.axis()
    .scale(xScale)
    .orient("bottom");

//定义y轴
var yAxis = d3.svg.axis()
    .scale(yScale)
    .orient("left");
```
x 轴刻度的方向向下，y 轴的向左。

## 4.4 添加矩形和文字元素

```
//矩形之间的空白
var rectPadding = 4;

//添加矩形元素
var rects = svg.selectAll(".MyRect")
        .data(dataset)
        .enter()
        .append("rect")
        .attr("class","MyRect")
        .attr("transform","translate(" + padding.left + "," + padding.top + ")")
        .attr("x", function(d,i){
            return xScale(i) + rectPadding/2;
        } )
        .attr("y",function(d){
            return yScale(d);
        })
        .attr("width", xScale.rangeBand() - rectPadding )
        .attr("height", function(d){
            return height - padding.top - padding.bottom - yScale(d);
        });

//添加文字元素
var texts = svg.selectAll(".MyText")
        .data(dataset)
        .enter()
        .append("text")
        .attr("class","MyText")
        .attr("transform","translate(" + padding.left + "," + padding.top + ")")
        .attr("x", function(d,i){
            return xScale(i) + rectPadding/2;
        } )
        .attr("y",function(d){
            return yScale(d);
        })
        .attr("dx",function(){
            return (xScale.rangeBand() - rectPadding)/2;
        })
        .attr("dy",function(d){
            return 20;
        })
        .text(function(d){
            return d;
        });
```
矩形元素和文字元素的 x 和 y 坐标要特别注意，要结合比例尺给予适当的值。

## 4.5 添加坐标轴的元素

```
//添加x轴
svg.append("g")
  .attr("class","axis")
  .attr("transform","translate(" + padding.left + "," + (height - padding.bottom) + ")")
  .call(xAxis);

//添加y轴
svg.append("g")
  .attr("class","axis")
  .attr("transform","translate(" + padding.left + "," + padding.top + ")")
  .call(yAxis);
```
## 4.6 完整效果图
![image](http://www.ourd3js.com/wordpress/wp-content/uploads/2014/06/5111.png)

# 5. 结语
使用D3 可以方便的实现各种可视化效果，它与其他可视化工具的主要区别如下：
![image](http://wiki.jikexueyuan.com/project/d3wiki/images/layout-2.png)

- 选择 D3：如果希望开发脑海中任意想象到的图表；
- 选择 Highcharts、Echarts 等：如果希望开发几种固定种类的、十分大众化的图表。
