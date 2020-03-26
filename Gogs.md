# Gogs

## 什么是Gogs

​	Gogs是一款极易搭建的自主Git服务

​	Gogs的目标是打造一个最简单的，最快速的最轻松的方式搭建自助Git服务。使用Go语言开发是的Gogs能够通过独立的二进制分发，并且支持Go语言支持的所有平台，包括Liunx，MacOSX，Windows以及ARM平台。

​	地址：https://gitee.com/Unknown/gogs

## Gogs安装与配置

### 安装

1. 下载镜像

   ```dockerfile
   docker pull gogs/gogs
   ```

2. 创建目录

   ```dockerfile
   mkdir -p /var/gogs
   ```

3. 创建容器

   ```dockerfile
   docker run --name=gogs -p 10022:22 -p 3000:3000 -v /var/gogs:/data gogs/gogs
   ```

### 配置

假设centos虚拟机IP为192.168.x.x，完成以下步骤

1. 在地址栏输入http://192.168.x.x:3000 会进入首次运行安装程序页面，我们可以选择一种数据库作为gogs数据的存储，最简单的是选择SQLite3。如果对于规模较大的公司可以选择MySQL，这步骤我选择SQLite3

   ![](D:\images\2020-2-26\20200226175522.jpg)

2. 注册账号，随便填写，上传项目要记住这些账号

   ![](D:\images\2020-2-26\20200226175712.jpg)

3. 登录

   ![](D:\images\2020-2-26\20200226175555.jpg)

4. 创建仓库，输入仓库名称就可以了，其它的可以自己选择

   ![](D:\images\2020-2-26\20200226175743.jpg)

   ![](D:\images\2020-2-26\20200226175852.jpg)

   ![](D:\images\2020-2-26\20200226175928.jpg)

## idea 配置 git

1. 首先本地要有git，在idea菜单添加git路径，File - settings，窗口选择Version Control - Git

   ![](D:\images\2020-2-26\20200226174901.jpg)

## 将项目提交到Git

1. 选择菜单VCS - Enable Version Control Integration

![](D:\images\2020-2-26\20200226180238.jpg)

![](D:\images\2020-2-26\20200226180254.jpg)

2. 设置远程地址：右键点击工程选择菜单 Git - Repository - Remotes

![](D:\images\2020-2-26\20200226180549.jpg)

![](D:\images\2020-2-26\20200226180707.jpg)

3. 右键点击工程选择菜单 Git - Add

![](D:\images\2020-2-26\20200226180757.jpg)

4. 右键点击工程选择菜单 Git - Commit Directory

![](D:\images\2020-2-26\20200226180910.jpg)

![](D:\images\2020-2-26\20200226180959.jpg)

5. 右键点击工程选择菜单 Git - Repository - Push

![](D:\images\2020-2-26\20200226181118.jpg)

![](D:\images\2020-2-26\20200226181142.jpg)

<img src='D:\images\2020-2-26\20200226181201.jpg' align='left' style=' width:300px;height:100 px'/>

6. 打开仓库查看项目

![](D:\images\2020-2-26\20200226181252.jpg)