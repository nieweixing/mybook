# gitbook简介

gitbook网站是一个简单的个人在线书籍网站，在这里可以把自己的文档整理成书籍发布出来，便于阅读。

gitbook网站：https://legacy.gitbook.com/

本文主要讲解在gitbook网站上发布一个书籍文档和使用gitbook提供的工具在本地开发一个书籍文档部署到自己的服务上

在此之前你需要会如下准备：

* 账号： github有账号，gitbook使用github账号注册
* git：代码管理工具
* Markdown：gitbook主要使用MD语法来编写书籍的
* gitbook工具：如果你在本地开发需要安装此插件，下面有介绍
* nodejs环境：gitbook插件需要的运行环境
* 一款Markdown编辑器：方便本地开发，推荐Typora或gitbook自己的编辑器gitbook editor

# 安装nodejs环境

本次操作都在windows系统上进行操作，nodejs的安装具体可以参考文档<https://www.runoob.com/nodejs/nodejs-install-setup.html>

# 安装gitbook

```
npm install gitbook-cli -g
```

# 初始化gitbook

创建一个文件夹gitbook，然后初始化gitbook，最后运行gitbook

```
# mkdir gitbook
# cd gitbook
# gitbook init
# gitbook serve 
```

运行后会在启动一个服务，可以在浏览器输入localhost:4000，这样就可以访问gitbook了