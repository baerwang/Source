# PostgreSql基本操作

1. ​	安装

   1.1 yum

   详见https://www.postgresql.org/download/linux/redhat

   yum -y install pgdg-centos96-9.6-3.noarch.rpm

   yum -y install postgresql96

   yum -y install postgresql96-server

   1.2 rpm

   cd ~/Downloads/postgresql/installation/9.6

   rpm -ivh *.rpm

2. 初始化

   /usr/pgsql-9.6/bin/postgresql96-setup initdb

3. 开机启动

   systemctl enable postgresql-9.6

4. 配置

   cd /var/lib/pgsql/9.6/data

   4.1 pg_hba.conf

   （1）光标定位至末尾处

   （2）将local all all ident改为local all all trust

   （3）将host all all 127.0.0.1/32 ident改为host all all 127.0.0.1/32 password

   （4）若是测试环境，则新增host all all 192.168.1.0/24 password

   4.2 postgresql.conf

   （1）将max_connections的值修改为1000

   （2）若是测试环境，将listen_addresses的值改为'*'，并解除注释

5. 命令

   5.1 启动

   systemctl start postgresql-9.6

   5.2 停止

   systemctl stop postgresql-9.6

   5.3 重启

   systemctl restart postgresql-9.6

   5.4 查看状态

   systemctl status postgresql-9.6 

6. 迁移数据库目录

   systemctl stop postgresql-9.6

   mv /var/lib/pgsql /datahome

   ln -s /datahome/pgsql /var/lib/pgsql

   chown -h postgres:postgres /var/lib/pgsql

7. 命令行

   7.1 进入数据库

   ​	su postgres → psql

   7.2 创建角色

   ​	create role root login createdb createrole superuser replication password 'datahome123';

   7.3 创建数据库

   ​	create database home owner root;

   7.4 查看角色

   ​	\du

   7.5 查看数据库

   ​	\l

   7.6 连接数据库

   ​	\c home

   7.7 查看表

   ​	\dt

   7.8 备份数据  是在/home目录下，不是在sql环境里

   ​	pg_dump -U postgres home -O -x > /datahome/home.bk

   7.9 恢复数据

   ​	psql -U root -d home < /datahome/home.bk

// 备份恢复 单表数据

数据库用户名：root

数据库：keirs

表明：registration_lookups

表备份输出的文件：/datahome/backup/registration_lookups.bk

备份：

pg_dump -U root keirs -t registration_lookups -f /datahome/backup/registration_lookups.bk

恢复：

psql -U root -d keirs < /datahome/backup/registration_lookups.bk

https://www.cnblogs.com/MakeView660/p/6133589.html 外网访问

```
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE datname='testdb' AND pid<>pg_backend_pid(); 
--- 注：testdb 替换成自己的数据库  session 链接不能操作数据库
```

8. pgbench

   详见：https://blog.csdn.net/enzesheng/article/details/42720691

   8.1 初始化表

   ​		create database pgbench;

   ​		pgbench -U postgres -h 127.0.0.1 -s 1 -I pgbench

   ​		-s：生成的行数乘以比例因子。比如，-s 100会在pgbench_accounts表中将会生成1000万条记录。默认值是1。

   8.2 执行压力测试

   ​	pgbench -U postgres -h 127.0.0.1 -T 10 -c 1 -r pgbench

   ​	pgbench -U postgres -h 127.0.0.1 -t 100 -c 1 -r pgbench

   ​	-c：模拟的客户数量,具体指并发运行的数据库会话数。默认值为1。

   ​	-t：每个客户端运行的事务数量，默认值为10。

   ​	-T：运行测试这么多秒,而不是每个客户端运行固定数量的事务。-t - t是互相排斥的。

   ​	-r：基准测试运行完成后报告每个命令的平均延迟时间（从客户角度看到的执行时间）。