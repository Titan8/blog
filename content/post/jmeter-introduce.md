+++
date = "2017-03-07T10:44:56+08:00"
title = "jmeter introduce"
menu = ""
draft = false
images = []
categories = ["test"]
tags = ["autotest"]
banner = ""
description = ""

+++
![image](/img/jmeter.png) 
#Jmeter  windows下安装步骤：

###一.安装JDK
【步骤一】安装jdk：

jdk下载地址：http://www.oracle.com/technetwork/java/javase/downloads/index.html

【步骤二】配置jdk环境变量

右键计算机属性->高级系统设置->系统属性->高级->环境变量->添加如下的系统变量：
变量名：【JAVA_HOME】
变量值：【D:\Program Files\Java\jdk1.8.0_112】【jdk安装路径】
变量名：【path】
变量值：【\;%JAVA_HOME%\bin;】
变量名：【CLASSPATH】
变量值：【.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;】【.不能漏】
![image](/img/path.jpg)
【步骤三】验证jdk

运行cmd->输入java -version->显示java版本就表示jdk安装成功，如下图
![image](/img/cmd.png)
###二.Jmeter安装

【步骤一】安装Jmeter
1、下载Jmeter，官网地址：http://jmeter.apache.org/download_jmeter.cgi
2、解压Jmeter安装包
【步骤二】配置Jmeter环境变量
按下面变量名和变量值配置Jmeter系统环境变量：
【变量名】JMETER_HOME
【变量值】F:\CYL\test\jmeter\apache-jmeter-3.0（根据实际的jmeter解压路径填写）
【变量名】CLASSPATH
【变量值】%JMETER_HOME\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\logkit-2.0.jar;
![image](/img/path1.png)

【步骤三】启动Jmeter
双击Jmeter解压路径（D:\Program Files\apache-jmeter-3.1\apache-jmeter-3.1\bin）的bin下面的jmeter.bat，如下图
![image](/img/runjmeter.png)
###三.Jmeter使用
Jmeter有两种运行模式，一种是GUI模式，GUI模式非常消耗资源，单个客户端测试无法达到目标压力。而使用非 GUI 模式，即命令行模式运行JMeter测试脚本能够大大缩减所需要的系统资源，使用远程启动模式也可以像Loadrunner那样进行分布式测试。
【步骤一】http协议接口测试
1.测试计划下新增线程组
2.线程组下新增http信息头管理器
名称：Content-Type 值：application/json
3.增加http请求和结果显示
4.运行测试
【步骤一】no-gui启动测试
1.DOS命令执行语法：jmeter -n -t <testplan filename> -l <listener filename>。
2.如果脚本在其他目录，并且执行结果存放到其他目录，需要使用绝对路径，cmd中如：jmeter -n -t D:\E\02_Test\A2_性能测试\JMeter\JMeter脚本\product_select.jmx -l D:\E\02_Test\A2_性能测试\JMeter\JMeter测试结果\product_select_20160311001.jtl1

###四.Badboy+Jmeter配合使用
【步骤一】badboy简介
Badboy是用C++开发的，被用于测试和开发复杂的动态应用。它提供了强大的屏幕录制和回放功能，同时也提供了丰富的图形结果分析功能。只要不用于商业目的就可以免费使用。因此这两工具的结合，就成为了绝配。我们可以用Badboy录制脚本，然后将录制的脚本导出为JMeter格式的脚本，最后将该脚本导入到JMeter，借助于JMeter强大的测试功能模拟大量的虚拟用户，进行复杂的性能测试。
其下载地址为：http://www.badboy.com.au。下载完后需要进行安装，安装过程同一般的Windows 应用程序没有什么区别，安装完成后你可以在桌面和Windows开始菜单中看到相应的快捷方式。如果找不到，可以找一下Badboy安装目录下的Badboy.exe文件，直接双击启动Badboy。最后开到的启动界面如下：
  
![image](/img/badboy.png)
                     
【步骤一】badboy使用方法
a.在地址栏（图中红色方框标注的部分）中输入你需要录制的Web应用的URL，这里我们以http://www.baidu.com为例。
b.点击“开始录制”按钮（图中蓝色圆圈标注的部分）开始录制 。
c.开始录制后，你可以直接在Badboy内嵌的浏览器（主界面的右侧）中对被测应用进行操作，所有的操作都会被记录在主界面左侧的编辑窗口中（图中黄色方框标注的部分）。在这个试验中，我们在baidu的搜索引擎中输入 JMeter 进行搜索。不过录制下来的脚本并不是一行行的代码，而是一个个Web对象——这有点像LoadRunner的VuGen中的Tree View视图。
d.录制完成后，点击工具栏中的“停止录制”按钮（图中紫色方框标注的部分），完成脚本的录制。
 
![image](/img/export.png)
 
e.选择“File -> Export to JMeter”菜单，填写文件名“baidu.jmx”，将录制好脚本导出为JMeter脚本格式。也可以选择“File -> Save”菜单保存为Badboy脚本。
f.启动JMeter并打开刚刚生成的测试脚本，就可以用JMeter进行测试了。
