# Kettle手册

## 安装

1. 需要安装JDK(1.8) [JDK-1.8安装教程](https://www.cnblogs.com/manmanchanglu/p/11783720.html)
2. 下载[Kettle](https://sourceforge.net/projects/pentaho/files/Data%20Integration/7.1/pdi-ce-7.1.0.0-12.zip/download)
3. 解压pdi-ce-7.1.0.0-12.zip
4. 双击运行Spoon.bat

![image-20210311093058678](images\image-20210311093058678.png)

![](images\image-1615426824.jpg)

## 从文本文件读出写到excel文件或浏览数据

### 1.新建一个转换工作

   <img src="images\image-20210311094355.jpg" style="zoom: 67%;" />

### 2.托动文本文件输入组件

   ![](images\image-1615427456.jpg)

### 3.在桌面或任何文件新建一个后缀txt文件并添加内容保存--间隔使用(Tab)键位输入

   ![](images\image-1615428516.jpg)

### 4.双击打开空白板的文本文件输入组件

   1. 打开浏览选择刚才编写的文件

   2. 添加、列表会出现文件相关的信息

   3. 选择导航栏的内容

         ![](images\image-1615428772.jpg)

   4. 在内容的分隔符和编码方式选择
         ![](images\image-1615429228.jpg)

   5. 选择导航栏的字段并获取字段关闭扫描结果点击确定

         ![](images\image-1615429757.jpg)

### 5.拖动文本文件输出组件

![](images\image-1615429995.jpg)

### 6.按shift键位点击文本文件输入到空操作

![](images\image-1615430091.jpg)

### 7.点击运行在点击空操作选择执行结果的Preview data 

点击运行前先保存文件

![](images\image-1615430568.jpg)

### 8.从文本文件输入写到excel文件

![](images\image-20210311104816.jpg)

### 9.按shift键位点击文本文件输入到Excel输出

### 10.点击Excel输出设置文件存放位置

![](images\images-1615431130.jpg)

### 11.选择导航栏字段获取字段确定-并运行

![](images\image-20210311105317.jpg)

### 12.查看写入的文件



## 转换机制

### 1.字符串替换

![](images\image-20210311110125.jpg)

### 2.点击字符串替换组件选择要替换的字段并输入值和要替换的内容

![](images\image-1615431824.jpg)

### 3.运行查看空操作的Preview data

![](images\image-20210311110610.jpg)

## 保存到数据库

### 1.拖动插入/更新组件

![](images\image-20210311111135.jpg)

### 3.在数据库创建表这里操作的是PostgreSql

如果要操作其他的数据库需要下载对应的数据库驱动包放到/lib/里面，重启Kettle

![](images\image-20210311111611.jpg)

### 2.点击插入/更新组件选择数据库

![](images\image-1615432709.jpg)

### 3.选择目标表、点击获取字段和要更新的字段，如果不更新可以不选择

![](images\image-20210311112027.jpg)

### 4.运行 查看数据库

![](images\image-20210311113201.jpg)

## 数据校验过滤机制

### 1.拖动数据校验组件

​	数据校验通过数据会进入通过浏览数据、数据校验不通过会进入不通过浏览数据

![](images\image-1615434676.jpg)

### 2.设置name字段不允许为空、school字段长度限制

1. 点击数据校验组件添加规则

   ![](images\image-20210311115343.jpg)

2. 选择要检验的字段

   ![](images\image-20210311115551.jpg)
   ![](images\image-20210311115708.jpg)

### 3.通过

![](images\image-20210311134903.jpg)

### 4.不通过

![](images\images-20210311135040.jpg)