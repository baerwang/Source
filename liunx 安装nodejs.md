# liunx 安装nodejs

### [安装包链接地址](https://npm.taobao.org/mirrors/node/v16.6.1/)

### 安装nodejs

#### 下载安装包并解压

```shell
wget https://npm.taobao.org/mirrors/node/v16.6.1/node-v16.6.1-linux-x64.tar.xz
xz -d node-v16.6.1-linux-x64.tar.xz
tar -xf node-v16.6.1-linux-x64.tar
```

#### 设置软连接，设置全局可使用该命令

```shell
sudo ln -s /opt/asants/node-v16.6.1-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /opt/asants/node-v16.6.1-linux-x64/bin/npm /usr/local/bin/npm
```

#### 重启，并检查nodejs是否安装成功

```shell
reboot
node -v
```

### 设置淘宝镜像，下载依赖库快

```shell
npm config set registry https://registry.npm.taobao.org
```

### 安装vue

```shell
ln -s /opt/asants/node-v14.1.0-linux-x64/bin/vue /usr/local/bin/vue
npm install -g vue-cli
vue --version
```
### 安装yarn

```shell
yum -y install yarn
yarn -v
```

