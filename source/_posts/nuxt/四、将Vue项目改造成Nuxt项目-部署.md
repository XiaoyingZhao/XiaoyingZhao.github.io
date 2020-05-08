---
title: 四、将Vue项目改造成Nuxt项目-部署
date: 2020-04-17 19:52:00
tags: 
  - javascript
  - nuxt
  - vue
categories:
  - 将Vue项目改造成Nuxt项目
---
### 正常部署流程
1. 本地项目完成之后，`npm run build`完成打包工作
2. 将项目上传至服务器
3. 在服务器执行`npm install`安装
4. 执行`npm run start`运行项目

### 问题
1. 在本地执行`npm run dev`运行没有问题，但是上传服务器打包之后运行不成功。
**解决：**
大多是因为有部分代码在服务端不被识别，如window等，建议在本地自己先执行`npm run start`差错
2. 在服务器安装速度比较慢，并且有可能出现网络差安装错误的情况，所以尽量不在服务端执行安装命令，需要将node_modules文件夹上传至服务器，这样又会延伸出一个问题： node_modules文件夹太大，上传时间慢，资源服务器压力大。
**解决：**
  a. node_modules只安装生产环境的依赖 `npm install --production`
  b. 将node_modules压缩
3. 如果每次上传都要删掉开发环境的依赖文件再去安装生产环境的依赖实现起来特别繁琐，很容易操作失误。
**解决：**
使用 `shelljs`插件编写一个shell脚本，每次需要修改依赖时自动执行脚本进行压缩node_modules文件夹
**实现：**
a. 编写`shell.js`脚本文件
  ```js
  var shell = require('shelljs');
  if (shell.exec('rm -rf install && mkdir install && cp package.json ./install && npm install --production --prefix ./install/ && tar -zvcf node_modules.tar.gz ./install/node_modules').code !== 0) {
    shell.echo('Error: ...');
    shell.exit(1);
  }
  /**
    这个脚本实现的功能是：
    1.如果有install这个文件夹，先删掉它 
    2.创建一个install文件夹 
    3.将package.json文件复制一份到install文件夹中 
    4.安装开发环境依赖到install中 
    5.将install/node_modules文件压缩成名为 node_modules.tar.gz的压缩文件
  **/
  ```
b. 运维配置软连接指向该文件即可
