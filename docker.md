# docker

centos 安装docker
步骤：
	1.检查内核版本，必须是3.10以上
		uname -r
	2.安装jdk，1.8以上版本
	3.安装docker
		yum install docker
		输入 y 确认安装
		最后一行显示 Complete!说明安装成功！
	4.启动docker
		systemctl start docker
		docker -v #显示docker版本
	5.关闭docker
		systemctl stop docker
	6.将docker 服务设为开机启动
		systemctl enable docker
	7.docker搭配阿里云镜像
		vim /etc/docker/daemon.json
		{
		    "registry-mirrors": ["https://0x3p2nkn.mirror.aliyuncs.com"]
		}
	8.重新启动服务
		systemctl daemon-reload
	9.重新启动docker
		systemctl restart docker