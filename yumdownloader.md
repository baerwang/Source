# yumdownloader

> ​	利用yumdownloader工具下载rpm包，离线安装需要的依赖包

```shell
yum -y install yum-utils
yum -y install gcc
yumdownloader --resolve --destdir=`pwd` gcc
yum -y localinstall *.rpm
```