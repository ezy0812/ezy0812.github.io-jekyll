---
title:      "CTFHub WriteUp 01-技能树中的网站源码WriteUp"
date:       2020-02-29 23:13:00
author:     "ezy"
categories: WriteUp
excerpt_separator: <!--more-->
tags:
    - CTFHub
---
![f1]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f1.PNG)
<!--more-->

**[题目链接](https://www.ctfhub.com/#/skilltree)**

## 题目给出提示
![f2]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f2.PNG)
## 利用burp交叉遍历所有目录
![f3]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f3.PNG)
### 得出结果
![f4]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f4.PNG)
## 访问该网址得到压缩文件
![f5]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f5.PNG)
### 压缩包里有3个文件
![f6]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f6.PNG)
## 没想到flag居然是访问txt的文件，得到flag
![f7]({{site.baseurl}}/assets/images/images_of_WriteUp/CTFHub_writeUp_01_f7.PNG)
