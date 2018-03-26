---
title: 用Hexo+github pages搭建Blog 的笔记(主要是Hexo的笔记)
category: blog笔记
tags: hexo
date: 2016-07-25
---


---
因为 github 远程仓库之前已经建好了 所以跳过github pages 仓库的创建
Hexo 初始化
### 1.安装Hexo
Hexo是基于NodeJS，所以需要先安装NodeJS

    $ npm install -g hexo-cli
### 2.初始化框架


    $ hexo init <yourFLoder>   // 初始化到你创建的那个Blog文件夹
    $ cd  <YourFloder>  // 切换到你创建的那个Blog文件夹

### 3.安装依赖
    $ npm install
### 4.初始化完成大概的目录

    .
    ├── _config.yml //网站的`配置`信息，您可以在此配置大部分的参数。
    ├── package.json
    ├── db.json // json格式的静态常量数据库    
    ├── node_modules // Hexo的功能JavaScript文件
    ├── public // 生成静态网页文件
    ├── scaffolds   //模版文件夹。当您新建文章时，Hexo会根据scaffold来建立文件。
    ├── source     //资源文件夹是存放用户资源的地方。
    |   ├── _drafts // 草稿文件夹
    |   └── _posts // 文章文件夹
    └── themes     //主题文件夹。Hexo会根据主题来生成静态页面。

### 5.新建文章（默认会创建一个Hello World）
    $ hexo new <title> for example : "Hello World"
在Blogs/source/_post里添加hello-world.md文件，之后新建的文章都将存放在此目录下.
如果要删除，直接在此文件夹下删除对应的文件即可
### 6.生成网站
    $ hexo generate   // 可以简写 $ hexo g
此时会将Blogs/source的.md文件生成道/public中，形成网站的静态文件
### 7.服务器
    $ hexo server // 在本地客户端查看
会出现以下内容

    INFO  Start processing
    INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
将**http://localhost:4000/** 复制到浏览器打开就可以预览到默认创建的那个hello world
### 8.部署网站
先在博客配置文件 _config.yml中配置你所要部署的站点,这边我是用github pages, 文档最下面有Deployment 只需在这边配置github pages 的信息(type 和 repo 和 branch)(repo的地址上你的github 去复制过来将好了 这边我是用https的 还有一种ssh 没试过,估计后者是不用输密码的~)

    # Deployment
    ## Docs: https://hexo.io/docs/deployment.html
    deploy:
      type: git
      repo: https://github.com/ChenPt/ChenPt.github.io.git
      branch: master
type那一项有的教程是写github 但是最新版的还是写git 所以写git就好了
配置好后,把博客deploy(部署)到github上 
    $ hexo deploy
此时如果没有意外的话,部署将成功了,打开浏览器输入  **<用户名>.github.io**将可以查看你的blog了
### 9.常用Hexo命令
    $ hexo c == hexo clean
执行此项操作会删除public文件夹中的内容 (也就是我们写的那些md文件)

    $ hexo g == hexo generate   // 将Blogs/source的.md文件生成道/public中，形成网站的静态文件
    $ hexo s == hexo server  //本地预览(此命令可以在你添加新文章后做预览用 看有没有生效~)
    $ hexo d == hexo deploy // 部署到你的站点上
