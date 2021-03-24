# PostgreSQL

[PostgreSQL官网](https://www.postgresql.org/)

## 安装

1. 下载PostgreSQL

   ```cmd
   yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   ```

2. 安装客户端软件包

   ```cmd
   yum install postgresql96-server
   ```

3. 安装服务器软件包(可选)

   ```cmd
   yum install postgresql96-server
   ```

4. 初始化数据库并启动自动启动，如果初始化失败的话，重新安装客户端软件包

   ```cmd
   /usr/pgsql-9.6/bin/postgresql96-setup initdb # 初始化数据
   systemctl enable postgresql-9.6 # 开机启动数据库
   systemctl start postgresql-9.6 # 启动数据库
   ```

5. 修改PostgreSQL用户密码

   ```cmd
   su - postgres # 切换用户，执行后提示符 会变成 -bash-4.2$
   psql -U postgres # 登陆数据库
   ALTER USER postgres WITH PASSWORD 'wangxiaohu'; # 设置postgres用户密码,密码要用双引号，要有分号
   \q # 退出数据库
   exit # 退出
   ```

6. 开启远程访问，修改 #listen_addresses = 'localhost'

   ```cmd
   vim /var/lib/pgsql/9.6/data/postgresql.conf
   ```

   内容

   ```cmd
   listen_addresses = '*'
   ```
   
7. 开启信任远程连接

   ```cmd
   vim /var/lib/pgsql/9.6/data/pg_hba.conf
   ```

   内容

   ```txt
   host    all             all             0.0.0.0/0               md5
   ```

   ![](D:\images\2020-2-26\20200303112930.jpg)

8. 重启postgresql

   ```cmd
   systemctl restart postgresql-9.6.service
   ```

9. 查看状态

   ```cmd
   systemctl status postgresql-9.6.service
   ```

10. 查看版本，安装成功

   ```cmd
   psql --version
   ```

## 基本使用

> 进入数据库：su postgresql
>
> 输入密码：psql
>
> 查看数据库：psql -l
>
> 创建角色：create role root login createdb createrole superuser replication password '123456'
>
> 查看角色：\du
>
> 创建数据库： create database home owner root
>
> 进入数据库：psql demo # demo是数据库名称
>
> 删除数据库：dropdb demo # demo是数据库名称
>
> 创建表：psql table test (title varchar(225),content text); # test是表名
>
> 查看表：\dt
>
> 查看表数据：\d test # test是表名
>
> 修改表名：alter table test rename to hello;  # test是当前表名，hello是修改后的表名
>
> 更改字段名：alter table hello rename title to titles; # hello是表名，title是字段，titles是修改后的字段名
>
> 删除表：drop table hello; 
>
> 导入数据库文件：\i db.sql 	# db.sql是自己创建的数据库文件，下面已经帮你写好了复制即可。
>
> 在-bash-4.2$ 目录下创建，也可以是其他的目录，自行百度
> 	vim db.sql , 内容位下方：
> 		create table posts(
>      title varchar(255) not null,
>      content text,
>      created_date timestamp default 'now'
> 	);

## 备份/恢复

备份数据 在目录下，不是在sql环境

```cmd
pg_dump -U postgres home -O -x > home.bk
```

恢复数据

```cmd
psql -U root -d home < /home.bk
```

备份恢复**单表**数据

备份

```cmd
pg_dump -U root db -t table -f /table.bk
```

恢复

```cmd
psql -U root -d db < /table.bk
```

## pgbench 压力测试工具

详见：https://blog.csdn.net/enzesheng/article/details/42720691

1. 初始化表

    create database pgbench;

    pgbench -U postgres -h 127.0.0.1 -s 1 -I pgbench

    -s：生成的行数乘以比例因子。比如，-s 100会在pgbench_accounts表中将会生成1000万条记录。默认值是1。

2. 执行压力测试

    pgbench -U postgres -h 127.0.0.1 -T 10 -c 1 -r pgbench

    pgbench -U postgres -h 127.0.0.1 -t 100 -c 1 -r pgbench

    -c：模拟的客户数量,具体指并发运行的数据库会话数。默认值为1。

    -t：每个客户端运行的事务数量，默认值为10。

    -T：运行测试这么多秒,而不是每个客户端运行固定数量的事务。-t - t是互相排斥的。

    -r：基准测试运行完成后报告每个命令的平均延迟时间（从客户角度看到的执行时间）。

## 字段类型

[参考网站](https://www.postgresql.org/docs/9.6/index.html)

[参考网站](https://www.postgresql.org/docs/9.6/datatype.html)

PostgreSql的基础数据类型

* 数值型：
  + integer(int)
  + real
  + serial
* 文字型：
  * char
  * varchar
  * text
* 布尔型：
  * boolean
* 日期型：
  * date
  * time
  * timestamp
* 特色类型：
  * Array
  * 网络地址型(inet)
  * JSON类
  * XML型

## 添加表约束

> create table test(
>         id serial primary key,
>         title varchar(255) not null,
>         content text check(length(content) > 3 ),
>         is_draft boolean default TRUE,
>         is_del boolean default FALSE,
>         created_date timestamp default 'now'
> );
>
> 约束条件
> not null：不能为空
> unique：在所有数据中值必须唯一
> check：字段设置条件
> default：字段默认值
> primary key(not null,unique)：主键，不能为空，且不能重复

## insert 插入

> insert into [tablename] (fieid,...) values (value,...);
>
> insert into test (title,content) values('postgres','helloworld');	# 其余的都是默认
>
> insert into test (title,content) values(NULL,'helloworld');		  # 其中字段title设置约束了不能为空，报错信息为 ERROR:  null value in column "title" violates not-null constraint
>
> insert into test (title,content) values('','helloworld');		  		 # 其中字段title可以设置为 ''
>
> content设置了约束必须大于3的长度，报错信息为 ERROR:  new row for relation "test" violates check constraint "posts_content_check"
>
> select * from test; 																	# 查询表的数据

## select 查询

> select * from test;					# 查询全部
>
> \x											# 这个作用是吧数据由横向显示编程纵向显示，是为了让我们看清每一个字。
> select * from test;
> \x											# 结束横向显示
>
> select random();					# 随机选择
>
> select title,content from test;  # 查询某个列的数据

## where 条件

> select * from posts where is_draft != false;
>
> select * from posts where is_draft != false;
>
> select * from posts where content  like 'w%';
>
> 这都不用介绍了 会sql就会，简单一逼

## 数据抽出选项

> select * from users order by id  asc;
>
> select * from users order by id desc;
>
> select * from test order by id desc limit 3;
>
> select * from test order by id desc limit 3 offset 3;
>
> 这都不用介绍了 会sql就会，简单一逼

## 统计抽出数据

> select distinct title from test; # 返回唯一不同的值
>
> select max(score) from users;
>
> select max(score) from users;
>
> select * from users where score = (select min(score) from users);
>
>  select * from users where score = (select max(score) from users);
>
> select team, max(score)from users group by team;
>
> select team, max(score)from users group by team having max(score) >= 25;
>
> select team, max(score)from users group by team having max(score) >= 25 order by max(score);
>
> 这都不用介绍了 会sql就会，简单一逼

## 方便的函数

> select player,length(player) from users;
>
>  select player,concat(player,'/',team) from users;
>
> select substring(team,1,2) as "首文字" from users;
>
>  select concat('xxss',substring(team,1,2)) as "首文字" from users;
>
> select * from users order by random():

## update 更新 和 delete 删除

> update users set score = 100 where player ='阿詹';
>
> update users set score = score + 1 where player ='阿詹';
>
> update users set score =score + 100  where team in ('勇士','湖人');
>
> delete from users where score > 30;
>
> delete from users where id = 1;
>
> 这都不用介绍了 会sql就会，简单一逼

## 变更表结构

> alter table users add fullname varchar(255);		# 给表添加字段
>
> alter table users drop fullname;							# 删除表中的字段
>
> alter table users alter fullname type varcahr(100);	# 改变表中的字段的类型
>
> create index fullname_index on users(fullname);	  # 添加索引
>
> drop index fullname_index;									# 删除索引

## 操作多个表

> select users.player , twitters.content from users, twitters where users.id = twitters.user_id;
>
> select u.player,t.c ontent from users as u,twitters as t where u.id =t.user_id;
>
> select u.player,t.content from users as u,twitters as t where u.id =t.user_id and u.id = 1;

## 视图

​	视图(view)是从一个或多个表导出的对象。视图与表不同，视图是一个虚拟表，即视图所对应的数据不进行实际存储，数据库只存储视图的定义，在对视图的数据进行操作时，系统根据视图的定义去操作与视图相关联的基本表。

​	视图就是一个select语句，把业务系统中常用的select语句简化成一个类似于表的对象，便于简单读取和开发。

> select * from test;											# 查询数据
>
> create view test_view as select * from test;	# 创建视图
>
> \dv																   # 查看视图
>
> \d test_view													# 查看视图的类型
>
> drop view test_view;									  # 删除视图

## 事务

​	数据库事务(Database Transaction)，是指作为单个逻辑工作单元执行的一系列操作，要么完全执行要么完全不执行。事务处理可以确保非事务性单元内的所有操作都成功完成，否则不会永久更新面向数据的资源。通过将一组相关操作组合为一个要么全部成功要么全部失败的单元，可以简化错误恢复并使应用程序更加可靠。一个逻辑工作单元要成为事务，必须满足所谓的ACID(A：原子性，C：一致性，I隔离性，D：持久性)。事务是数据库运行中的逻辑工作单位，由DBMS中的事务管理子系统负责事务的处理。

> 事务使用

			- begin(开始)，commit(提交)，rollback(回滚)

> ​	begin;
>
> ​	select * from test;
>
> ​	update test set title  = '张三' where id = 1;
>
> ​	select * from test;
>
> ​	commit或rollback;