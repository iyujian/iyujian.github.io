---
layout: post
title:  "IntelliJ IDEA提高开发效率的八条配置"
date:   2018-12-20 17:15:15 +0800
categories: others
tags: [IntelliJ IDEA]
---
1. 自动编译开关
2. 忽略大小写开关
3. 智能导包开关
4. 悬浮提示开关
5. 取消单行显示tabs的操作
6. 项目文件编码
7. 滚轴修改字体大小
8. 设置行号显示

## 1. 自动编译开关

File > Settings > Build, Execution, Deployment > Compiler > Build project automatically，勾选。

![自动编译开关](../upload/2018/12/20/2018-12-20_1.jpg "自动编译开关")

## 2. 忽略大小写开关

IntelliJ IDEA默认是匹配大小写，此开关如果未关。你输入字符一定要符合大小写。比如你敲string是不会出现代码提示或智能补充。但是，如果你开了这个开关，你无论输入String或者string都会出现代码提示或者智能补充！

File > Settings > Editor > General > Code Completion > Case sensitive completion， 选择 “None”。

![忽略大小写开关](../upload/2018/12/20/2018-12-20_2.jpg "忽略大小写开关")

## 3. 智能导包开关
File > Settings > Editor > General > Auto Import > Insert imports on paste / Add unambiguous imports on the fly / Optimize imports on the fly (for current project)。

![智能导包开关](../upload/2018/12/20/2018-12-20_3.jpg "智能导包开关")

## 4. 悬浮提示开关

打开这个开关后。只要把鼠标放在相应的类上，就会出现提示。

File > Settings > Editor > General > [Other] Show quick documentation on mouse move，勾选。

![悬浮提示开关](../upload/2018/12/20/2018-12-20_4.jpg "悬浮提示开关")

## 5. 取消单行显示tabs的操作

取消勾选后，打开多个文件的时候，会换行显示，非常直观，大大提升效率！

File > Settings > Editor > General > Editor Tabs > Show tabs in single row，取消勾选。

![取消单行显示tabs的操作](../upload/2018/12/20/2018-12-20_5.jpg "取消单行显示tabs的操作")

## 6. 项目文件编码

File > Settings > Editor > File Encodings > Global Encoding / Project Encoding / Default encoding for properties files / Transparent native-to-ascii conversion

Transparent native-to-ascii conversion：自动转换ASCII编码。

他的工作原理是：在文件中输入文字时他会自动的转换为Unicode编码，然后在idea中发开文件时他会自动转回文字来显示。
这样做是为了防止文件乱码。这样你的properties文件，一般都不会出现中文乱码！

![项目文件编码](../upload/2018/12/20/2018-12-20_6.jpg "项目文件编码")

## 7. 滚轴修改字体大小

勾选后，按住Ctrl+滚轴可以修改编辑器字体大小。

File > Settings > Editor > General > Change font size (Zoom) with Ctrl+Mouse Wheel，勾选。

![滚轴修改字体大小](../upload/2018/12/20/2018-12-20_7.jpg "滚轴修改字体大小")

## 8. 设置行号显示

File > Settings > Editor > General > Appearance > Show line numbers，勾选。

![设置行号显示](../upload/2018/12/20/2018-12-20_8.jpg "设置行号显示")

