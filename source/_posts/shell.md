---
title: 编写shell脚本
date: 2019-07-31
category: shell
tags: [shell, 脚本]
---

# 为何要编写shell脚本

之前在使用ubuntu的时候写过node的命令行脚本去启动ssr，但是使用node去写脚本需要安装的依赖比较多，比如shelljs，还有一些读取命令行参数的插件。而使用bash来写的话，是不需要安装这些依赖的，所以这次用原生shell，具体是用`bash`。

# 知识储备

## echo命令

`echo`的作用就是输出字符串

### echo 重定向

`echo 'hello world' > file` 输出字符串到file文件里，如果file文件已经存在，会直接覆盖。
`echo 'hello world' >> file` 输出字符串追加到file文件的后面 

## read命令
读取用户输入，然后将输入放入变量中。

# 实现

## 实现思路
具体的实现思路就是获取用户的输入，再将用户输入重定向输出到文件中，再将文件移动到博客资源目录下，最后用vscode打开该文件进行编写。所以需要使用`read`命令获取用户输入，使用`echo`命令输出一些字符串提示用户，`echo >` 或者 `echo >>`将字符串重定向输出到指定文件中，使用`mv`命令移动文件，使用`code`命令用vscode打开文件。

## 实现代码
``` bash
#/bin/bash
# author: ChenPt

# 输入文件名字
echo "input the fileName:"
read file_name

file=${file_name}.md
echo "---" > $file
# 输入文章标题
echo "input the title"
read title
echo "title: ${title}" >> $file

# 设置时间
time=$(date "+%Y-%m-%d")
echo "date: ${time}" >> $file

# 输入仓库名称
echo "input the category"
read category
echo "category: ${category}" >> $file

# 输入标签
echo "input the tags"
read tags
echo "tags: [$tags]" >> $file
echo "---" >> $file
mv $file source/_posts
code source/_posts/$file
echo "the end"
```
