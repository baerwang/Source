# webpack

- 安装node.js
- 检查环境变量是否成功
- node -v
- npm-v
- npm配置淘宝镜像:npm config set registry https://registry.npm.taobao.org

## 创建webpack

1. 新建项目
2.  npm init 初始化项目，使用npm管理项目中的包
3.  安装webpack：全局安装：npm i webpack -g。局部安装：npm i webpack --save-dev  
4.  安装webpack-cli  npm i --save-dev webpack-cli -g
5.  Webpack 入口文件路径 输出文件路径 对main.js处理
6.  打包：webpack 目标文件 -o 打包路径
7.  上面的方式缺点：每次修改代码的时候都需要进行打包，不太友好
8.  项目根目录下创建webpack.config.js，写入配置
9.  安装webpack-dev-server  npm i webpack-dev-server --save-dev
10.  在package.json的script节点下配置：”dev”:”webpack-dev-server”
11.  执行npm run dev
12.  安装html-webpack-plugin npm i html-webpack-plugin --save-dev