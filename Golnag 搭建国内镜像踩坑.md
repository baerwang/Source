# Golnag 搭建国内镜像踩坑

###### 在Windows系统下，go语言已经搭建好我们不在讲解，傻瓜式安装就好。

### 搭建国内镜像

###### 打开此电脑属性，选择高级系统属性，环境变量设置

![image-20200723142531236](C:\Users\baerw\AppData\Roaming\Typora\typora-user-images\image-20200723142531236.png)

![image-20200723142552382](C:\Users\baerw\AppData\Roaming\Typora\typora-user-images\image-20200723142552382.png)

###### 新增用户变量，我们添加两个环境

输入变量名：GO111MODULE
输入变量值：on
![image-20200723142733180](C:\Users\baerw\AppData\Roaming\Typora\typora-user-images\image-20200723142733180.png)

输入变量名：GOPROXY
输入变量值：https://goproxy.cn
![image-20200723142918515](C:\Users\baerw\AppData\Roaming\Typora\typora-user-images\image-20200723142918515.png)

#### 完成搭建国内镜像了，安装快的起飞