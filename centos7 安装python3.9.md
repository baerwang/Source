# centos7 安装python3.9

### 下载脚本安装

```shell
wget "https://repo.anaconda.com/miniconda/Miniconda3-py39_4.9.2-Linux-x86_64.sh"
```

### 执行脚本

```shell
bash Miniconda3-py39_4.9.2-Linux-x86_64.sh
回车
yes
/opt/asants/conda3 # 指定路径，如果不指定路径就直接回车
```

### 重启服务器

```shell
reboot
```

### 查看是否安装成功

```shell
python3 -V
python -V
```

### 设置一些参数

```shell
conda config --set auto_activate_base true
/opt/asants/conda3/bin/conda config --set auto_activate_base true
```

### 部署项目前相关需要执行的命令

```shell
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

yum install -y centos-release-scl

yum install -y  https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

yum install -y postgresql13-devel

export PATH=/usr/pgsql-13/bin/:$PATH

yum install -y gcc

yum install -y python3-devel

pip3 install -r requirements.txt
```

