---
title: hexo使用mathjax书写公式
category: hexo
tags: [mathjax, hexo, netlify, CDN]
updated: 2018-4-18
date: 2018-4-18
---
为了这玩意，搞了一晚上。不记录都觉得很难受。

<!-- more -->

## 关于LaTex和MathJax
写浮点数的总结的时候，为了可以写公式，我了解到了LaTeX这东西，这东西主要是用来排版的。

而想在hexo博客中书写公式之类的东西，需要使用MathJax这个JS库将LaTeX公式转化为[MathML](https://developer.mozilla.org/zh-CN/docs/Web/MathML)。MathML是W3C提出的。

对于NexT主题来说，可以使用MathJax，不过默认是没开启的，需要去theme文件夹下的_config文件启用MathJax。还有就是每篇使用到mathjaxd的文章开头都需要填写`mathjax: true`
``` 
---
title: xxx
mathjax: true
---
```

``` yml
# MathJax Support
mathjax:
  enable: true
  per_page: false
  cdn: https://cdn.staticfile.org/MathJax/MathJax-2.6-latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```
其他主题的话需要安装`hexo-math`，可以参考github这项目的介绍，或者这个[网站](http://catx.me/2014/03/09/hexo-mathjax-plugin/)。
不过还是使用NexT的默认配置来得直接痛快点。

一开始我这个CDN地址使用的是NexT主题默认的CDN，这个URL是没有添加`https://`在前面的，这是一个坑...。

开启这个MathJax之后，我用hexo的命令构建和在本地服务器上预览，发现整体上没多大问题，然后当我提交修改到github仓库后，Netlify部署的博客里的所有LaTeX都给我原封不动的解析成文字，我又反复尝试其他办法，卸载hexo默认的markdown渲染引擎，修改为可以渲染LaTeX语法的一个扩充的引擎，还有就是安装`hexo-math`这个npm包，其实都没多大卵用，对于Netlify还是没有真正使用这个MathJax。

我去到`hexo-math`的github仓库的issue里面找，发现了有个人说，MathJax之前的CDN托管将会关闭,需要更新这个CDN，同一个issue里，另外一个人说，对于使用https协议的静态网站来说，所使用的cdn地址需要显式地添加`https://`，修改config文件的cdn的地址之后，重新用Netlify部署，发现一切正常，大喊一声...FQ,然后就结束了这个hexo使用MathJax的踩坑之路。真是坑爹阿.

## 关于LaTex的简单语法

### 行内写法
行内写法使用一对`$...$`，如`$f(n) = a + b$`

效果如右： $f(n) = a+b$



### 块式写法
块式写法使用两对`$$...$$`，如`$$K=2^n$$`。

$$K=2^n$$

中间公式跟首尾美元符之间最好不要有空格。有可能渲染效果会有差异。
还有就是对于公式内的特殊符号，与markdown语法冲突的，需要用`\`来转义。(这里又很蛋疼了，我用VSCode来写md，装了个LaTex预览插件，是不用去转义的，所以在VSCode上预览的跟在博客上预览的又不一样了...)

如`$$K=x_a$$` 需要写成`$$K=x\_a$$`

$$K=x\_a$$

如果不想要使用转义，


### 起始范围
使用花括号来表示起始范围

``` LaTeX
$$y=x^{2n+1}$$
```
$$y=x^{2n+1}$$

## THE END
Thanks https://github.com/hexojs/hexo-math/issues/41

非常感谢感谢这个大兄dei.我自己估计问题是出现在Netlify上，真不知道原来是这个mathjax的CDN和HTTPS之间的原因。感谢..

更多LaTeX公式和写法参考
https://www.zybuluo.com/Cesar/note/228458

.




