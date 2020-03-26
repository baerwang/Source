# Dockerfile

## 简介

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新镜像。

- 开发人员：可以为开发团队提供一个完全一致的开发环境
- 测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个的镜像开始工作了
- 运维人员：在部署时，可以实现应用的无缝移植

## 常用命令

| 命令                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| FROM image_name:tag                | 定义了使用哪个基础镜像启动构建流程                           |
| MAINTAINER user_name               | 声明镜像的创建者                                             |
| ENV key value                      | 设计环境变量(可以写多条)                                     |
| RUN command                        | 是Dockerfile的核心部分(可以写多条)                           |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压                      |
| WORKDIR path_dir                   | 设置工作目录                                                 |
| EXPOSE port1 port2                 | 用来指定端口，使容器内的应用开通过端口和外界交互             |
| CMD argument                       | 在构建容器时使用，会被docker run后的argument覆盖             |
| ENTRYPOINT argument                | 和CMD相似，但是并不会被docker run指定的参数覆盖              |
| VOLUME                             | 将本地文件夹或者其他容器的文件挂载到容器种                   |

![](D:\images\2020-2-19\20200226103059.jpg)

## 使用脚本创建镜像

1. 创建目录

   ```dockerfile
   mkdir -p /usr/local/dockerjdk8
   ```

2. 下载jdk-8u161-linux-x64.tar.gz上传到虚拟机中的 /usr/local/dockerjdk8

   ![20200225153951](D:\images\2020-2-19\20200225153951.PNG)

3. 创建文件Dockerfile

   ```cmd
   vim Dockerfile
   ```

   ```txt
   #依赖镜像名称和ID
   FROM centos:7
   #指定镜像创建者信息
   MAINTAINER SOCIAL
   #切换工作目录
   WORKDIR /usr
   RUN mkdir /usr/local/java
   #ADD 是相对路径jar,把java添加到容器中
   ADD jdk-8u161-linux-x64.tar.gz /usr/local/java/
   #配置java环境变量
   ENV JAVA_HOME /usr/local/java/jdk1.8.0_161
   ENV JRE_HOME $JAVA_HOME/jre
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
   ENV PATH $JAVA_HOME/bin:$PATH
   ```

   ![](D:\images\2020-2-19\20200225153951.PNG)

4. 执行命令构建镜像

   ```dockerfile
   docker build -t='jdk1.8' .
   ```

   ![](D:\images\2020-2-19\20200225154258.PNG)

5. 查看镜像是否建立完成

   ```dockerfile
   docker images
   ```

6. 创建容器

   ```dockerfile
   docker run -it --name=myjdk8 jdk1.8 /bin/bash
   ```

7. 输入java-version，出了环境配置就没问题了，exit退出

   ```cmd
   java -version
   ```

8. 查看容器

   ```dockerfile
   docker ps
   ```


## 运行简单的demo

1. 随便找个demo打包成jar

2. 放在虚拟机的任何目录

3. 创建Dockerfile 不带后缀，要和当前放入的jar包在一起的目录

   ```dockerfile
   vim Dockerfile
   ```

   内容

   ```dockerfile
   FROM java:8	# jdk8,安装
   MAINTAINER "BAERWANG"	# 作者名称
   VOLUME /tmp	# 指定的目录
   ADD app.jar exam.jar # app.jar 指的是你的jar包名称,exam.jar 创建jar包的名称
   EXPOSE 9066	# 端口
   ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/exam.jar"] # 运行环境
   ```

4. 创建镜像

   ```dockerfile
   docker build -t exam .
   ```

5. 查看镜像

   ```dockerfile
   docker images # 查看镜像,里面有exam.jar 就说明创建镜像成功
   ```

6. 运行镜像

   ```dockerfile
   docker run -d --rm --name social_config -p 9066:9066 exam:latest
   ```

7. 如果要查看日志的话

   ```dockerfile
   docker logs -f --tail=500 social_config # social_config 容器的名称
   ```

8. 打开浏览器访问，ip(虚拟机地址):指定的端口

# Idea集成Docker实现镜像打包一键部署

## Docker开启远程访问

1. 修改该docker服务文件

   ```dockerfile
   vim /lib/systemd/system/docker.service
   ```

2. 修改ExecStart

   ```txt
   -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock 
   ```

   ![](D:\images\2020-2-19\20200226105538.jpg)

3. 刷新配置，重启服务

   ```cmd
   systemctl daemon-reload
   systemctl restart docker
   ```

4. 查看端口是否开启

   ```cmd
   netstat -nlpt
   ```

5. curl查看是否生效

   ```cmd
   curl http://192.168.x.x:2375/info # 虚拟机地址
   ```

## Idea安装Docker插件

1. Idea默认帮我安装好了，没有的话就ctrl+alt+a 打开plugins安装，安装好重启

![](D:\images\2020-2-19\20200226110220.jpg)

2. 添加ip地址和阿里云镜像

   ![](D:\images\2020-2-19\20200226110642.jpg)

   ![](D:\images\2020-2-19\20200226110833.jpg)

3. idea下方工具栏有docker按钮，如果没有找到从services里面找，连接docker

   ![](D:\images\2020-2-19\20200226111759.jpg)

4. 运行docker容器，创建images镜像，自己慢慢研究去吧

## Docker-Maven-Plugin

传统过程中，打包，部署，上传到Liunx，编写Dockerfile，构建镜像，创建容器运行

而在持续集成过程中，项目工程一般使用Maven编译打包，然后生成镜像，通过镜像上线，能够大大提供上线效率，同时能够快速动态扩容，快速回滚，着实很方便。Docker-maven-Plugin插件就是为了帮助我们在maven工程中，通过简单的配置，自动生成镜像并推送到仓库中。

微服务部署有两种方法：

1. 手动部署：基于源码打包生成jar包或war包，将jar包或war包上传至虚拟机并拷贝至JDK容器

2. 通过maven插件自动部署

   对于数量众多的微服务，手动部署无疑是非常麻烦的做法，并且容易出错误。

我们用DockerMaven插件来完成打包上传镜像步骤

1. pom.xml

   ```xml
   <build>
           <!-- 为当前工程起名称 -->
           <finalName>app</finalName>
           <plugins>
               <!-- spring boot的maven插件 -->
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
               <plugin>
                   <groupId>com.spotify</groupId>
                   <artifactId>docker-maven-plugin</artifactId>
                   <version>1.2.1</version>
                   <configuration>
                       <!-- 做成镜像后镜像的名称 -->
                       <imageName>${project.build.finalName}</imageName>
                       <!-- 基础镜像-->
                       <baseImage>java</baseImage>
                       <!--制作者提供信息-->
                       <maintainer>baerwang baerwang@126.com</maintainer>
                       <!-- 执行命令，打jar包 -->
                       <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                       <!-- copy the service's jar file from target into the root directory of the image -->
                       <resources>
                           <resource>
                               <targetPath>/</targetPath>
                               <!--表示target目录-->
                               <directory>${project.build.directory}</directory>
                               <!--用于指定需要复制的文件,${project.build.finalName}.jar 指的是打包后的jar文件-->
                               <include>${project.build.finalName}.jar</include>
                           </resource>
                       </resources>
                       <!--指定远程docker.api地址-->
                       <dockerHost>http://192.168.1.106:2375</dockerHost>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   ```
   
2. 在win命令提示符，进入social_eureka工程所在的目录，输入命令，进行打包并构建镜像到Docker上

   1. 把项目拉近到terminal就可以进入提示符了就不需要在重新打开另外的提示符了

   ![](D:\images\2020-2-19\20200226134900.jpg)

   2. 执行命令

   ```cmd
   mvn clean package docker:build
   ```

   3. 构建完毕的结果

   ![](D:\images\2020-2-19\20200225184132.jpg)

3. 查看镜像或者idea插件查看

   ```cmd
   docker images
   ```

   ![](D:\images\2020-2-19\20200226132448.jpg)

4. 运行镜像，docker时区和宿主机时区是不相同的，需要设置 -e TZ 为当前时间同步，

   ![](D:\images\2020-2-19\20200226132541.jpg)

   ```dockerfile
   -p 9066:9066 -e TZ="Asia/Shanghai"
   ```

   

5. 打开浏览器输入，ip:端口，ip是你自己的虚拟机，端口是你刚才设置的开放的端口

# Docker 私有仓库

## 私有仓库搭建与配置

1. 拉取私有仓库镜像

   ```dockerfile
   docker pull registry
   ```

2. 启动私有仓库容器

   ```dockerfile
   docker run -di --name=registry -p 5000:5000 registry
   ```

3. 打开浏览器输入地址 http://192.168.x.x:5000/v2/_catalog 看到 {repositories: [ ]}，表示私有仓库搭建成功并内容为空

4. 修改daemon.json，添加内容，此步用于让 docker信任私有仓库地址 

   ```cmd
   vim /etc/docker/daemon.json
   ```
   
   如果配的有阿里云用  ,  隔开

   ```txt
     "insecure-registries": [
       "192.168.x.x:5000"
     ]
   ```

5. 重启docker 服务

   ```dockerfile
   systemctl restart docker
   ```

## 像上传至私有仓库

1. 标记此镜像为私有仓库的镜像

   ```dockerfile
   docker tag jdk1.8 192.168.x.x:5000/jdk1.8
   ```

   ![](D:\images\2020-2-19\20200225164049.PNG)

2. 启动私服容器

   ```dockerfile
   docker start registry
   ```

3. 上传标记的镜像

   ```dockerfile
   docker push 192.168.x.x:5000/jdk1.8
   ```

   ![](D:\images\2020-2-19\20200225164342.PNG)

4. 再次打开浏览器输入地址 http://192.168.x.x:5000/v2/_catalog，查看是否上传成功

# 理解持续集成

## 什么是持续集成

​	持续集成	Continuous integration，简称CI

​	随着软件开发复杂度的不断提高，团队开发成员间如何更好低协同工作以确保软件开发的质量已经慢慢成为开发过程中不可回避的问题。尤其是近些年来，敏捷(Agile)在软件工程领域越来越红火，如何能再不断变化的需求中快速适应和保证软件的质量也显得尤其的重要。

​	持续集成正是针对这一类问题的一种软件开发实践。它倡导团队开发成员必须经常集成他们的工作，甚至每天都可能发生多次集成。而每次的集成都是通过自动化的构建来验证，包括自动编译，发布和测试，从而尽快地发现集成错误，让团队能够更快的开发内聚的软件

## 持续集成的特点

- 它是一个自动化的周期性的集成测试过程，从检出代码，编译构建，运行测试，结果记录，测试统计等都是自动完成的，无需人工干预。
- 需要有专门的集成服务器来执行构建。
- 需要有代码托管工具来支持。

## 持续集成作用

- 保证团队开发人员提交代码的质量，减轻了软件发布时的压力。
- 持续集成中任何一个环节都是自动完成的，无需太多的人工干预，有利于减少重复过程以节省时间，费用和工作量。

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

2. 创建容器

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

# 运用Jenkins实现持续集成

## Jenkins简介

​	Jenkins，原名Hudson，2011年改为现在的名字，它是一个开源的实现的持续集成的软件工具。[官网网站](http://jenkins-ci.org/) 

​	Jenkins能实施监控集成中存在的错无，提供详细的日志文件和提醒功能，还能用图表的形式形象低展示项目构建的趋势和稳定性。

特点：

- 易安装：一个java -jar jenkins.war，从官网下载该文件后，直接运行，无需额外的安装，更无需安装数据库
- 易配置：提供友好的GUI配置界面
- 变更支持：Jenkins能从代码仓库(Subversion/CVS)中获取并产生代码更新列表并输出到编译输出信息中
- 支持永久连接：用户是通过web来访问Jenkins的，而这些web页面的连接地址都是永久链接地址，因此你可以再各种文档中直接使用该链接
- 集成E-Mail/RSS/IM：当完成一次集成，可通过这些工具实时告诉你集成结果(据我所知)，构建一次集成需要花费一定的是时间，有了这个功能，你就可以再等待结果过程中，干别的事情
- JUnitTestNG测试报告：也就是用图标等形式提供详细的测试报表功能
- 支持分布式构建：Jenkins可以把集成构建等工作分发到多台计算机中完成
- 文件指纹信息：Jenkins会报错哪些集成构建产生了哪些jars文件，哪一次集成构建使用了那个版本的jars文件等构建记录
- 支持第三方插件：使得Jenkins变得越来越强大

## Jenkins安装

### Jenkins安装与启动

1. [下载Jenkins](https://pkg.jenkins.io/redhat-stable/)

2. 安装jenkins

   ```cmd
   rpm -ivh jenkins-2.204.1-1.1.noarch.rpm
   ```

3. 修改jenkins的配置，因为它默认的jdk你的目录下不一定符合，需要修改

   ```cmd
   vim /etc/init.d/jenkins
   ```

   ![](D:\images\2020-2-19\20200226221349.jpg)

4. 配置jenkins，修改用户和端口

   ```cmd
   vim /etc/sysconfig/jenkins
   ```

   ```txt
   JENKINS_USER="root"
   JENKINS_PORT="8888"
   ```

5. 刷新配置，启动服务

   ```cmd
   systemctl daemon-reload
   systemctl start jenkins
   ```

6. 打开浏览器访问链接 http://192.168.x.x:8888 

7. 如果你打开的时候，出现Please wait while Jenkins is getting ready to work页面需要修改配置换国内镜像

   1. 查看rmp 解压的文件路径在哪里

      ```cmd
      rpm -qpl jenkins-2.204.1-1.1.noarch.rpm
      ```

   2. 修改配置

      ```cmd
      vim /var/lib/jenkins/hudson.model.UpdateCenter.xml
      ```

   3. 把里面的http://删掉换国内镜像

      ```txt
      https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
      ```
      
   4. 修改default.json，把里面的路径换一下

      ```cmd
      cd /var/lib/jenkins/update
      vim default.json
      ```
   ```
      
      ```cmd
      :1,$s/http:\/\/www.google.com/https:\/\/www.baidu.com/g
      :1,$s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g
   ```

8. 如果不知道密码，页面显示的有获取密码方式，把密码复制输入

   ```cmd
   cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

9. 选择安装社区推荐插件时如果出现 No such plugin: cloudbees-folder，要下载cloudbees-folder插件，选择对应的版本下载

   [cloudbess-folder](https://updates.jenkins-ci.org/download/plugins/cloudbees-folder/)
   
   复制到 /usr/lib/jenkins/jenkins.war/WEB-INF/datached-plugins/ 目录下

## Jenkins插件安装

### 安装Maven插件

1. 点击左侧的系统管理菜单然后点击管理插件
2. 选择可选插件选项卡，搜索Maven Integration，点击安装

### 安装Git插件

1. 有的git插件已经默认被安装了，需要自己查看，步骤如上面

## 全局工具配置

### 安装Maven与本地仓库

1. 下载 [maven](https://archive.apache.org/dist/maven/maven-3/3.6.1/binaries/)

2. 将Maven压缩包上传至服务器

3. 解压

   ```cmd
   tar zxvf apache-maven-3.6.1-bin.tar.gz
   ```

4. 移动目录

   ```cmd
   mv apache-maven-3.6.1 /usr/loca/maven
   ```

5. 修改setting.xml 配置文件，配置本地仓库目录和阿里云镜像

   ```cmd
   vim /usr/local/maven/apache-maven-3.6.1/conf/settings.xml
   ```

   ```xml
   <localRepository>/usr/local/repository</localRepository>
   ```

   ```xml
   <mirror>
   	<id>nexus-aliyun</id>
   	<mirrorOf>central</mirrorOf>
   	<name>Nexus aliyun</name>
   	<url>http://maven.aliyun.com/nexus/content/groups/public</url>
   </mirror>
   ```

6. 修改setting.xml

   pluginGroups 作用：

   ​	当插件的组织id(groupId)没有显示提供时，供搜寻插件组织Id(groupId)的列表。

   ​	该元素包含一个pluginGroup元素列表，每个子元素包含了一个组织的Id(groupId)。

   ​	当我们使用某个插件，并且没有在命令行为其提供组织Id(groupId)的时候，Maven就会使用该列表。

   ```xml
   <pluginGroups>
   	<pluginGroup>com.spotify</pluginGroup> 
   </pluginGroups>
   ```

7. 加入环境变量

   ```cmd
   vim /etc/profile
   ```

   ```cmd
   echo "export PATH=$PATH:/usr/local/maven/apache-maven-3.6.1/bin" >> /etc/profile
   ```

8. 让配置生效

   ```cmd
   source /etc/profile
   ```

9. 查看版本

   ```cmd
   mvn -v
   ```

### 全局工具配置

选择系统管理，全局工具配置

1. JDK配置

   设置别名为JDK，JAVA_HOME 为你当前的jdk位置

   ![](D:\images\2020-2-19\20200227163126.PNG)

2. Git配置

   如果本地有git这步骤省略，jenkins帮我们自动默认好了

   1.  [下载Git](https://mirrors.edge.kernel.org/pub/software/scm/git/)

   2. 将Git压缩包上传至服务器

   3. 安装所需软件包

      ```cmd
      yum  install autoconf
      yum install gcc
      ```

   4. 解压

      ```cmd
      tar zxvf git-2.9.5.tar.gz
      ```

   5. 进入目录配置编译安装

      ```cmd
      cd git-2.9.4
      make configure
      ./configure --prefix=/usr/local/git ##配置目录
      make profix=/usr/local/git
      make install
      ```

   6. 加入环境变量

      ```cmd
      vim /etc/profile
      ```

      ```txt
      echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
      ```

   7. 让配置生效

      ```cmd
      source /etc/profile
      ```

   8. 查看版本

      ```cmd
      git --version
      ```

3. Maven配置

   设置别名为Maven，Maven_HOMW 为你当前的maven位置

   ![](D:\images\2020-2-19\20200227163052.PNG)

## 持续集成

### 创建任务

1. 回到首页，点击新建按钮，输入名称，选择任何项目，我选择的是maven项目，这里是我部署的工作，点击ok

2. 源码管理，选择git

   ![](D:\images\2020-2-19\20200227223211.PNG)

3. 输入父工程的子项目的pom.xml

   ![](D:\images\2020-2-19\20200227223417.PNG)

   这里的Goals and options可以不输入，我是为了项目上传docker镜像的，用于清除、打包，构建docker镜像

4. 点击保存

### 执行任务

返回首页，再列表中找到我们刚才创建的任务

![](D:\images\2020-2-19\20200227223909.PNG)

点击右边的绿色牵头按钮，执行此任务

点击下面正在执行的任务，点击控制台就可以看见到输出的日志，正在下载包，服务器没有依赖的包，需要下载

![](D:\images\2020-2-19\20200227224104.PNG)

可以看到实时输出的日志

![](D:\images\2020-2-19\20200227224612.PNG)

构建完成

![](D:\images\2020-2-26\20200228001933.jpg)