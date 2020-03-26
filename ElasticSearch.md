# ElasticSearch

## 简介

Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速度去处理大规模数据。ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜素引擎，基于RESTFUL web接口。ElasticSearch使用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能用达到实时搜索，稳定，可靠，快速，安装使用方便。

## 特点

- 可以作为一个大型分布式集群(数百台服务器)技术，处理PB级数据，服务大公司，也可以运行在单机上。
- 将全文检索，数据分析及分布式技术，合并在了一起，才形成了独一无二的ES。
- 开箱即用，部署简单。
- 全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理。

## 体系结构

下表是Elasticsearch与MySQL数据库逻辑结构概念的对比 

| ElasticSearch  | 关系型数据库MySQL |
| -------------- | ----------------- |
| 索引(index)    | 数据库(databases) |
| 类型(type)     | 表(table)         |
| 文档(document) | 行(row)           |

## 安装

[ElasticSearch 下载](https://www.elastic.co/cn/downloads/elasticsearch) 

解压，打开cmd 进入es的bin目录输入elasticsearch，es的默认端口是9200，打开网页输入localhost:9200就说明运行成功。

## Postman调用Api

### 新建文档

以post方式提交   http://127.0.0.1:9200/apiindex/api

 在body以json格式下写

```json
{
	"title":"Hello World",
	"content":"你好 ElasticSearch"
}
```

返回结果

```json
{
    "_index": "articleindexs",
    "_type": "articles",
    "_id": "ksm1W3ABxRYEzGuEkdyV",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

_id是由系统自动生成的。 为了方便之后的演示，我们再次录入几条测试数据。

### 查询全部文档

查询某索引某类型的全部数据，以get方式请求 http://127.0.0.1:9200/apiindex/api/_search

![](D:\images\2020-2-19\20200219123516.PNG)

### 修改文档

以 put 形式提交，http://127.0.0.1:9200/apiindex/api/lsm3W3ABxRYEzGuE9dxo

api后面的那个是es的_id，填写body的json格式

```json
{
	"title":"es",
	"content":"张三"
}
```

返回结果

```json
{
    "_index": "apiindex",
    "_type": "api",
    "_id": "lsm3W3ABxRYEzGuE9dxo",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```

如果我们在地址中的_id不存在，则会创建新文档

查询修改的文档结果

![](D:\images\2020-2-19\20200219124046.PNG)

如果要设置_id，就按照刚才的put方式去提交，id设置自己的要求，id不存在会新建文档，这里我就不操作了。

### 按ID查询文档

按get方式提交 http://127.0.0.1:9200/apiindex/api/1

![](D:\images\2020-2-19\20200219124721.PNG)

### 基本匹配查询

根据某列进行查询，按照get方式提交

![](D:\images\2020-2-19\20200219125130.PNG)

### 模糊查询

可以用*代表任意字符

按get方式提交 http://127.0.0.1:9200/apiindex/api/_search?q=title:* a *，这个字符之间不用带空格，我这里是写的markdown格式，不会显示出来的，所以我要这样子写markdown格式。

### 删除文档

根据id删除文档，删除id为1的文档，按delete方式提交 http://127.0.0.1:9200/apiindex/api/1

![](D:\images\2020-2-19\20200219125627.PNG)

再次查看全部文档是否还存在该记录

## Head插件的安装与使用

如果通过rest请求的方式来使用es，未免太过于麻烦了，也不够人性化。我们一般都会使用图形化界面来实现es的日常管理，最常用的就是Head插件

### 安装

[elasticsearch-head-master.zip 下载](https://github.com/mobz/elasticsearch-head) 

解压到任意目录，不要安装es的目录

安装node.js，配淘宝镜像，这里不再讲了

将grunt安装为全局命令。Grunt是基于Node.js的项目构建工具。它可以自动运行你所设定的任务。

```cmd
npm install -g grunt-cli
```

打开 es-head的目录，在命令符提示符下输入命令，安装依赖

```cmd
npm install
```

进入es-head，在在命令符提示符下输入命令，启动head

```cmd
grunt server
```

默认端口是9100，输入http://127.0.0.1:9100

点击连接集群

![](D:\images\2020-2-19\20200219141551.PNG)

连接没有反应，需要跨域处理，es不允许跨域调用，es-head是属于前端的工程，会报错误

需要修改es的配置，让es允许跨域访问。

修改es的conf目录下的elasticsearch.yml，增加以下两句命令

```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

此步为允许es跨域访问，点击链接可以看到相关的信息

![](D:\images\2020-2-19\20200219144059.PNG)

### 新建索引

![](D:\images\2020-2-19\20200219165933.PNG)

### 新建文档

在复合查询中提交地址，输入内容，提交方式为PUT

![](D:\images\2020-2-19\20200219170826.PNG)

点击数据浏览，点击要查询的索引名称，右侧窗格中显示文档信息

![](D:\images\2020-2-19\20200219170916.PNG)

再次回到刚才的界面，修改数据提交请求，此时id已经存在，执行的是修改的操作。重新查询此记录发现版本为2，也就是说每次修改后的版本都会增加1。

![](D:\images\2020-2-19\20200219171144.PNG)

### 搜索文档

![](D:\images\2020-2-19\20200219172010.PNG)

### 删除文档

![](D:\images\2020-2-19\20200219172514.PNG)

## 分词器

### 什么是ID分词器

在Postman 地址输入http://127.0.0.1:9200/_analyze，body写json格式

![](D:\images\2020-2-19\20200219173826.PNG)

默认的中文分词是将每个字看成一个词，这显然是不符合要求的，所以我们需要安装中文分词来解决这个问题。

IK分词是一款国人开发的相对简单的中文分词器。虽然开发者自2012年之后就不再维护了，但是在工程应用中IK算是比较流行的一款！

### IK分词器安装

[elasticsearch-analysis-ik-7.6.0.zip 下载](https://github.com/medcl/elasticsearch-analysis-ik/releases) 

- 新建文件名为ik，把下载好的zip解压到ik文件里面
- 将ik文件夹拷贝到elasticsearch/plugins目录下
- 重新启动，即可加载ik分词器

### IK分词器测试

IK提供了两个分词算法ik_smart和ik_max_work，其中ik_smart 为最少切分，ik_max_word为最细粒度划分

- 最小切分，在Postman，以post方式提交

![](D:\images\2020-2-19\20200219175844.PNG)

- 最细切分，以post方式提交

![](D:\images\2020-2-19\20200219180109.PNG)

### 自定义词库

默认的分词并没有识别'奥立给'是一个词。如果我们想让系统之别"奥立给"是一个词，需要编辑自定义词库。

![](D:\images\2020-2-19\20200219180934.PNG)

进入elasticsearch/plugins/ik/config目录

新建my.dic文件，编辑内容 "老八秘制小汉堡" 和 "奥立给"   一行一个词

修改改IKAnalyzer.cfg.xml(在ik/config目录下)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">my.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```

重新启动es

以post方式提交

![](D:\images\2020-2-19\20200219181515.PNG)

## Logstash

### 简介

Logstash是一款轻量级的日志搜索处理框架，可以方便的把分散的，多样化的日志搜集起来，并进行自定义的处理，然后传输到指定的位置，比如某个服务器或者文件。

### 安装 -测试

[logstash-6.8.6.zip 下载](https://www.elastic.co/cn/downloads/past-releases#logstash) 

解压进入bin目录

```cmd
logstash -e "" 
```

控制台输入字符随后有日志输出

![](D:\images\2020-2-19\20200220161029.PNG)

- stdin：表示输入流，指从键盘输入
- stdout：表示输出流，指从显示器输出
- "" 默认为stdin，stdout

命令行参数

-e 执行

--config或-f配置文件，后跟着参数类型可以是一个字符串的配置或者全路径文件名或全路径(如：/etc/logstash.d/，logstash会自动读取/etc/logstash.d/目录下所有的*.conf的文本文件，然后在自己内存里拼接成一个完整的大配置文件再去执行)

## MySQL数据导入Elasticsearch

- 在logstash安装目录下创建文件下mysqletc

- 文件下创建mysql.conf，内容如下：

  ```
  input {
    jdbc {
  	  # mysql jdbc connection string to our backup databse
  	  jdbc_connection_string => "jdbc:mysql://192.168.1.106:3306/social_article?characterEncoding=utf8&useSSL=false&serverTimezone=UTC"
  	  # the user we wish to excute our statement as
  	  jdbc_user => "root"
  	  jdbc_password => "root"
  	  # the path to our downloaded jdbc driver  
  	  jdbc_driver_library => "E:\es6.8\logstash-6.8.6\mysqletc\mysql-connector-java-5.1.48.jar"
  	  # the name of the driver class for mysql
  	  jdbc_driver_class => "com.mysql.jdbc.Driver"
  	  jdbc_paging_enabled => "true"
  	  jdbc_page_size => "5000"
  	  #以下对应着要执行的sql的绝对路径。
  	  #statement_filepath => ""
  	  statement => "SELECT id,title,content,state FROM tb_article"
  	  #定时字段 各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新（测试结果，不同的话请留言指出）
        schedule => "* * * * *"
    }
  }
  
  output {
    elasticsearch {
  	  #ESIP地址与端口
  	  hosts => "127.0.0.1:9200" 
  	  #ES索引名称（自己定义的）
  	  index => "social"
  	  #自增ID编号
  	  document_id => "%{id}"
  	  document_type => "article"
    }
    stdout {
        #以JSON格式输出
        codec => json_lines
    }
  }
  ```

 - 将mysql启动包mysql-connector-java-5.1.48.jar拷贝到logstash/mysqletc下

 - 把mysql.conf上面的内容修改你的内容，比如账号密码，mysql启动包

 - 命令行执行

   ```cmd
   logstash -f ../mysqletc/mysql.conf
   ```

   再次刷新es-head的数据显示，看看是否也更新了数据，如果没有反应请按 enter键

## docker 安装es

- 下载镜像

```dockerfile
docker pull elasticsearch:6.8.6
```

- 创建容器

```dockerfile
docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:6.8.6
```

- 浏览器输入地址 http://192.168.x.x:9200/ 
- 修改demo的application.yml

```yml
spring:
  data:
    elasticsearch:
      cluster-nodes: 192.168.x.x:9300
      cluster-name: docker-cluster
```

运行程序发现会报错误

这是因为es从5版本以后默认不开启远程链接，需要修改配置文件

- 进入容器

```cmd
docker exec ‐it es /bin/bash
```

此时我们看到了es所在的目录为/usr/share/elasticsearch/，进入config看到了配置文件 **elasticsearch.yml**

输入 rpm  -qa|grep  vim 测试vim是否正确安装，会返回下面的代码：

```cmd
vim-enhanced-7.4.629-6.el7.x86_64
vim-minimal-7.4.629-6.el7.x86_64
vim-filesystem-7.4.629-6.el7.x86_64
vim-X11-7.4.629-6.el7.x86_64
vim-common-7.4.629-6.el7.x86_64
```

- 如果少了就在命令行上重新安装一遍

```cmd
yum -y install vim*
```

修改es.yml是允许任何ip地址访问es，开发测试阶段可以这么做，生产环境下制定具体的ip，如果不修改的话会报以下的错误

![](D:\images\2020-2-19\20200220183241.PNG)

- 修改es.yml

vim elasticsearch.yml，输入下面两行代码

```cmd
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
```

- 重新启动es

```cmd
docker restart es
```

重启动后发现启动失败了，这与我们修改的配置有，因为es在启动的时候回进行一些检查，比如最多打开的文件的个数以及虚拟内存区域数量等等，如果你放开了此配置，意味需要打开更多的文件以及虚拟内存，所以还需要系统调优。

- 系统调优

```cmd
vim /etc/security/limits.conf
```

输入以下两行代码

```txt
* soft nofile 65536
* hard nofile 65536
```

nofile是单个进程允许打开的最大文件个数，soft nofile 是软限制 hard nofile是硬限制

修改/etc/sysctl.conf，追加内容

```cmd
vim /etc/sysctl.conf
```

输入以下一行代码

```txt
vm.max_map_count=655360
```

限制一个进程可以拥有的VMA(虚拟内存区域)的数量

执行下面命令修改内核参数马上生效

```cmd
sysctl -p
```

重新启动虚拟机，再次启动容器，发现已经可以启动并远程访问 

## docker 安装IK分词器

把win系统安装的es里面的plugins 复制一份放到 /opt/里面

在主机将ik文件夹考本到容器内  /usr/share/elasticsearch/plugins 目录下

```cmd
docker cp /opt/ik/ es:/usr/share/elasticsearch/plugins/
```

- 重新启动，可加载分词器

```cmd
docker restart es
```

- 在Postman，以post方式提交，查看自定义的分词已经完成

![](D:\images\2020-2-19\20200220200447.PNG)

## Head插件安装

- 修改/usr/share/elasticsearch.yml ,添加允许跨域配置，在安装es的时候我们已经配置过了我们这里就不需要在修改了。
- 下载head镜像

```cmd
docker pull mobz/elasticsearch-head:5
```

- 启动head

```cmd
docker run -d --name=myhead -p 9100:9100 docker.io/mobz/elasticsearch-head:5
```

- 在浏览器中打开elasticsearch-head页面，填入ElasticSearch地址192.168.x.x/9100

