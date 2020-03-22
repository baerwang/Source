# Nginx

## 安装

1. 安装 **pcre** 依赖 

   1.  下载依赖包 http://downloads.sourceforge.net/project/pcre/pcre
   2. 拉取到liunx系统里
   3. 解压文件  tar -zxvf pcre-8.37.tar.gz
   4. ./configure 完成后，回到prce目录下执行，最后make install
   5. 查看依赖是否安装成功   pcre-config --version

2. 安装 **openssl 、zlib 、 gcc 依赖**

   ```liunx
   yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
   ```

3. 安装 **nginx**

   1. 下载依赖包 http://nginx.org/en/download.html
   2. 拉取到liunx系统里
   3. 解压文件 tar -zxvf nginx-1.14.2.tar.gz
   4. 进入 nginx-1.14.2目录，输入./configure 和 make && make install
   5. 进入/usr/local/nginx/sbin/nginx目录 输入 ./nginx 启动服务 
   6. 输入ps -ef| grep nginx 查看 nginx 启动状态
   7. 在浏览器输入 ip地址访问nginx，默认访问端口为80，可以在conf/nginx.conf 修改listen 的端口

## 常用命令

使用nginx操作命令提前条件必须进入nginx目录

1. 查看nginx版本号：./nginx -v
2. 启动nginx：./nginx
3. 停止nginx：./nginx -s stop
4. 重新加载nginx：./nginx -s reload

## 配置文件

vim /usr/local/nginx/conf/nginx.conf

1. 配置文件中的内容(包含三部分)

   1. 全局块： 配置服务器整体运行的配置指令

      从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令，主要包括配置运行nginx服务器的用户(组)，允许生成worker process 树，进程PID存放路径，日志存放路径和类型以及配置文件的引入等。

      比如上面的第一行配置

      ```txt
      worker_processes  1;
      ```

      这是nginx服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会收到硬件，软件等设备的制约。

   2. events块

      比如上面配置

      ```txt
      events {
          worker_connections  1024;
      }
      ```

      events块涉及的指令主要影响nginx服务器与用户的网络连接，常用的设置包括是否开启对多 work process下的网络连接进行序列化，是否允许同时接受多个网络连接，选取哪种事件驱动模型来处理连接请求，每个work process 可以同时支持的最大连接数等。

      上述例子就表示每个 work process 支持的最大连接数为1024

      这部分的配置对nginx的性能影响较大，在实际中应该灵活配置。

   3. http块

      ```txt
      http {
          include       mime.types;
          default_type  application/octet-stream;
      
          #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
          #                  '$status $body_bytes_sent "$http_referer" '
          #                  '"$http_user_agent" "$http_x_forwarded_for"';
      
          #access_log  logs/access.log  main;
      
          sendfile        on;
          #tcp_nopush     on;
      
          #keepalive_timeout  0;
          keepalive_timeout  65;
      
          #gzip  on;
      
          server {
              listen       80;
              server_name  localhost;
      
              #charset koi8-r;
      
              #access_log  logs/host.access.log  main;
      
              location / {
                  root   html;
                  index  index.html index.htm;
              }
      
              #error_page  404              /404.html;
      
              # redirect server error pages to the static page /50x.html
              #
              error_page   500 502 503 504  /50x.html;
              location = /50x.html {
                  root   html;
              }
          }
      }
      ```

      这算是 nginx 服务器配置中频繁的部分，代理，缓存和日志定义等绝大多数功能和第三方模块的配置都在这。

      需要注意的是：http块也可以包括http全局块，server块。

      1. http全局块

         http全局块配置的指令包括文件引入，MIME-TYPE定义，日志自定义，连接超时时间，单链接请求数上限等。

      2. server块

         这块和虚拟主机又密切关系，虚拟主机从用户角度来看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。

         每个http块可以包括多个server块，而每个server块就相当于一个虚拟机。

         而每个server块也分为全局server块，以及可以同时包含多个location块。

         1. 全局server块

            最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或IP配置。

         2. location

            一个server块可以配置多个location块。

            这块的主要作用是基于 nginx 服务器接受到的请求字符串(例如server_name/uri_string)，对虚拟主机名称(也可以是IP别名)之外的字符串(例如前面的/uri-string)进行匹配，对特定的请求进行处理。地址定向，数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。

## 配置实例 - 反向代理(一)

  运行tomcat，打开浏览器输入ip:端口 访问tomcat查看效果。

  在虚拟机进入 nginx conf目录，编辑ngxin.conf

```txt
      server {
          listen       80;
          server_name localhost;
  
          #charset koi8-r;
  
          #access_log  logs/host.access.log  main;
  
          location / {
              root   html;
              proxy_pass http://localhost:9090;	//这是tomcat的端口
              index  index.html index.htm;
          }
```

  在浏览器输入 ip地址访问nginx，发现nginx转发到tomcat的主页了。

## 配置实例 - 反向代理(二)

 使用 nginx 反向代理，根据访问的路径跳转到不同端口的服务中

  访问 http://ip/edu/ 直接跳转到 127.0.0.1:9090

  访问 http://ip/vod/ 直接跳转到 127.0.0.1:9091

  准备两个tomcat服务器，9090，9091端口，这里我用的是docker搭建

  准备一个页面里面不同的内容，把页面挂载到docker 容器里面

  分别是：

 ```dockerfile
  docker cp a.html CONTAINER ID:/usr/local/tomcat/webapps/edu
  
  docker cp a.html CONTAINER ID:/usr/local/tomcat/webapps/vod
 ```

修改 nginx配置文件，进行反向代理配置

```txt
	server {
          listen       80;
          server_name localhost;
  
          #charset koi8-r;
  
          #access_log  logs/host.access.log  main;
  
          location / {
              root   html;
              proxy_pass http://localhost:9090;	//这是tomcat的端口
              index  index.html index.htm;
          }
  
          location ~ /edu/ {
                  proxy_pass http://localhost:9090;	//这是tomcat的端口
          }
  
          location ~ /vod/ {
                  proxy_pass http://localhost:9091;	//这是tomcat的端口
          }
         }
```

  在浏览器输入 ip地址/vod/a.html 和 ip地址/edu/a.html，发现nginx转发到指定的的页面了。

## 负载均衡



