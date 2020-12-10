---
layout: post
title: SVN的忽略功能
categories:
  - 软件编程
tags:
  - 《道 · Coder’s work》
note: SVN的忽略功能
---

# SVN的忽略功能
## 一、IDEA中支持SVN
1. [IDEA中使用SVN的官方文档](https://www.jetbrains.com/help/idea/using-subversion-integration.html)  

## 二、IDEA中配置SVN
1. 在IDEA中使用SVN，需要安装SVN命令行，安装TortoiseSVN或者VisualSVN就能附带这个命令行工具，也可以单独下载。

2. [项目配置](https://www.cnblogs.com/yeminglong/p/6702449.html)

## 三、SVN配置忽略的文件
1. 使用IDEA自定义SVN配置的功能，找到或更改全局忽略配置的位置及内容。
![](https://i.loli.net/2019/12/19/E4NsdBPoaHpktDj.png)

2. 使用TortoiseSVN的全局忽略配置功能，找到或更改全局忽略配置的位置及内容。
![image.png](https://i.loli.net/2019/12/19/HQUDCwsrNmXfpgL.png)

3. IDEA提交时通过刷新来使最新的忽略配置生效
![image.png](https://i.loli.net/2019/12/19/XcZypN7SqO1lb3F.png)

## 四、将误加入SVN控制的文件取消版本控制
（摘取自Tortoise官方文档）

> 如果你不小心添加了一些应该被忽略的文件，你如何将它们从版本控制中去除而不会丢失它们？或许你
> 有自己的IDE配置文件，不是项目的一部分，但将会花费很多时间使之按照自己的方式工作。
> 如果你添加了文件但是没有提交, 你只需使用TortoiseSVN → Undo Add...来取消添加. 然后将其加入忽
> 略列表以免下次又错误添加.
> 如果文件已经在版本库里,它们就必须从版本库删除并添加到忽略列表.幸运的是TortoiseSVN有一个方便
> 的快捷方式来做这个工作. 运行TortoiseSVN → Unversion and add to ignore list将会标记file/
> folder为从版本库中删除, 但保留本地副本.它也会将这些文件自动加入忽略列表.然后提交父文件夹就
> 可以了。
