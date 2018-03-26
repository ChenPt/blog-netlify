---
title: 给自己的博客加个小绿锁
category: HTTPS
tags: [Blog, HTTPS]
date: 2018-03-27
---

既然学习了SSL协议，那么就顺便给自己的博客站点加个小绿锁吧，顺便迁移个博客。

<!-- more -->
## 探索之旅

一开始是准备用`Cloudflare`的，不过因为Cloudflare需要更改DNS服务器，而我的某个二级域名被捞仔们拿去开发招新网页了，修改DNS服务器会让整个招新网页域名解析错误，所以我就放弃了这个方案。

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqlq2inwmj20q005vt96.jpg)

`Cloudflare`实际上是个CDN服务商，其中开启SSL有个Flexible选项。开启后实际上当用户访问站点时，访问的内容是从CloudFlare的CDN服务器传输过来的，之间的传输采用的就是HTTPS协议，信息是加密的，而CDN与源服务器（github Page）的传输用的是普通的HTTP协议。

官网也告知说，要使用CloudFlare，需要去域名提供商那里更改DNS服务器，变为它提供的域名服务器地址。喜闻乐见，我改完DNS后，NW的招新网站就炸了，打扰了。
而且CloudFlare的访问速度也很慢，很慢，很慢。

使用`tarcert`来跟踪IP数据报的访问路径。发现我们的访问路径，从中国到了部署在欧洲的CloudFlare服务器再到达美国的github服务器。好吧，打扰了。

我继续科学搜索，找到了一个很好用的静态资源托管平台Netlify。



## 什么是Netlify？
它是一个提供静态资源网络托管的综合平台。

简单来说： 它的功能之一就跟我们之前Hexo博客的静态托管平台 `Github Page`一样, 不过，Netlify可比github Page功能多多了，而且速度也快。两者的对比在[netlify官网](https://www.netlify.com/github-pages-vs-netlify/)有介绍


简单来说它可以
* 托管静态资源
* 将静态网站部署到CDN上
* Continuous Deployment 持续部署，当你提交改变到git 仓库，它就会自动运行build command，进行自动部署。
* 可以添加自定义域名
* 重头戏：**可以启用免费的TLS证书，启用HTTPS**

> www.netlify.com 官网有更详细的文档内容。

## 开始前的准备，创建新仓库

之前的那个` yourname.github.io `的仓库继续保留着，可以不用删。 

我们新建一个仓库`blog-netlify`，不要添加README.md还有其他的一些东西，就创建个空仓库就好，不然后续步骤可能会因此报错。

然后我们在我们的博客目录下（.../../hexo）进行git的初始化

``` shell
git init  
```
我们需要将hexo生成的静态文件给忽略掉，还有node_modules，还有其他一些文件。

将忽略文件添加到`.gitignore`文件， 可以使用CLI的方式添加，或者手动创建文件，直接输入需要忽略的文件。
``` shell
echo "/public" >> .gitignore
echo "/node_modules" >> .gitignore
echo "/.deploy_git" >> .gitignore
echo "/.vscode" >> .gitignore
```
查看当前的工作目录和暂存区状态，如果还看到一些我们忽略的文件出现，则添加到`.gitignore`中。
``` shell
git status
```
将文件修改添加到暂存区中，并提交实际改动到分支上
``` shell
git add .

git commit -m "[F]Hello Netlify"
```
绑定远程仓库，输入刚才我们创建的仓库的地址
``` shell
git remote add origin <your repo url>  
```
查看绑定状态
``` shell
git remote -v
```
没什么问题就push到远程分支

``` shell
git push origin master
```


## 建立新站点

好了，一个netlify需要用到的仓库就搞定了，然后到[netlify官网](https://app.netlify.com/) 用github账户登录

点击页面的新建站点按钮

选择Github，

还有就是下面给netlify提供github的权限，这里需要勾上

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqmw0jfzvj20v50howg8.jpg)

选择你的博客仓库.

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqmyvqkbyj20qw0kqmzb.jpg)


Build command 填入 ** Hexo g **  Publish directory 填入 ** public **
![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqn28zthfj20s20nqdhl.jpg)

点击部署站点，它就会开始部署，稍等一下可以看到它的log日志，没什么问题的话，会显示成功部署。

接下来你可以通过它提供的二级域名`xxxx.netlify.com`来进行访问你的站点，你也可以自定义域名，添加自定义域名，先添加不带www的域名作为主域名，它会自动添加一个`www.domain.cc`重定向到`domain.cc`
这时它会检查你添加的两个域名，域名服务器来检查是否可以解析你的自定义域名，现在当然是不可以的，需要你去添加解析记录。

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnedheqij20vf0gdwfu.jpg)

通常需要去你的域名提供商那里添加两条解析记录，一条A记录，一条CNAME记录

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnderwn9j211903774f.jpg)
到这里就可以 通过自定义域名访问了，然后就是使用HTTPS了。

## 使用HTTPS
`Netlify` 使用的是 `Let’s Encrypt Certificate.`的免费证书，我这里因为我自己之前在腾讯云免费申请过证书了，所以这里就使用我自己的证书.

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnn4ffhoj209e04xgll.jpg)

第一项需要填的是你当前站点的安全证书，就是上图的chenpt.cc.crt

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnr06ck2j20ev045aag.jpg)

第二项是对应的私钥

第三项是中级证书，就是根证书下来的那个证书，就是上图的root_bundle.crt

打开证书文件你可以看到证书路径如下

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnq9hwtij20ee03qq3b.jpg)

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnme20fij20l10l9q3w.jpg)

使用的证书跟Apache一样，所以Netlify内部可能使用是Apache来配置HTTPS模块的。

输入完毕加载证书。 然后再启用强制HTTPS。
只能使用HTTPS协议的URL来访问站点，使用HTTP协议的URL会自动重定向HTTPS协议的URL，内部实现是返回301状态码 Move permanently（永久性移动）

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fpqnsnxjmbj20xo08ydgi.jpg)

## 开始新博客之旅

现在你想发布博客还需要hexo g -d 么？

答案是不用了。

现在是在source文件夹内的_posts文件添加新文章。然后把仓库的改变提交到分支上，再提交到远程分支上，`Netlify`获取仓库改变，会自动部署。 所以现在是很方便了。

而且netlify的速度比github page快了很多，体验nice！

之前的github page仓库还是留着吧，还有就是需要去把之前对github page的域名解析记录给删掉。然后就拥抱netlify吧.

希望大家都有小绿锁.