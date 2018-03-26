---
title: Ubuntu使用SS的方法总结
category: Ubuntu
tags: [Ubuntu, Shadowsocks]
date: 2017-07-17
---

# Ubuntu安装使用ss的方法.

前些日子想在Ubuntu上搞SS翻墙用下谷歌,在网上找了好多方法..(如下载SS-qt5版客户端等)都没搞定...所以索性回到windows系统学习和开发了...
暑假的某一天晚上又想用回ubuntu,这次下载了SS python版的客户端  
https://github.com/breakwa11/shadowsocks-rss/wiki/Python-client-setup-(Mult-language)  
在红莲的的帮忙下,实现了我去墙外了解世界的梦想.(滑稽.jpg); 下面就是安装和配置步骤了..

以上是摘要  

<!-- more -->


## 获取源代码(安装SS,python客户端)
首先是装git,我已经装了git的,所以这步骤跳过.  
新建一个文件夹,进入命令行界面,执行代码
``` bash 
git clone -b manyuser https://github.com/shadowsocksr/shadowsocksr.git

```
进入子目录:
``` bash 
cd shadowsocksr/shadowsocks
```

我这里是通过配置文件运行的,所以我先写好配置文件ss.json,并把配置文件放到此目录下..
配置文件写入以下内容:  
**注意**: json文件不能有注释代码...

```
{
    "server":"0.0.0.0",
    "server_ipv6": "::",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "udp_timeout": 60,
    "method":"aes-256-cfb",
    "protocol": "auth_aes128_md5",
    "protocol_param": "",
    "obfs":"http_simple",
    "obfs_param": "",
    "fast_open": false,
    "workers": 1
}
```
一般需要更改的有 服务器地址,端口号,密码,加密方式,使用协议.

然后就运行这个配置文件,命令如下  
后台运行:  

``` bash 
python local.py -c ./ss.json -d start
```
为了方便我把此命令写入be.sh文件方便调用. 如下  
``` bash
vim be.sh
```
*be.sh*
``` bash 
python local.py -c ./ss.json -d start
```
:wq 保存退出  

给be.sh 增加权限,修改be.sh为可执行文件 (chmod +x filename)
``` bash
chmod +x be.sh
```
然后运行be.sh文件
``` bash 
./be.sh
```
此时ss已经开启,墙外的世界已经连上了...

然后使用google搜索,不出意外,应该可以成功使用.  

## 配置PAC代理
然后就是配置PAC代理了

**安装genpac**  
这是基于gfwlist的代理自动配置Proxy Auto-config)文件生成工具,支持自定义规则..  
选择生成文件的存放位置进入命令行,我是放在~/etc/里面的  
执行命令

``` bash 
sudo pip install genpac
```
进入生成文件的位置
``` bash 
cd ~/etc/vpnPAC

sudo genpac --proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
```
然后可以看到生成了一个文件autoproxy.pac  
执行pwd命令查看文件所在路径待会需要用到
```
pwd
```
我这里是/home/chenpengteng/etc/vpnPAC/

### 将代理应用到整个系统

系统设置=>网络=>网络代理  
"方法"选择**自动**,"配置URL"填写 file:///home/chenpengteng/etc/vpnPAC/autoproxy.pac  
点击"应用到整个系统",整个步骤将完成了,此时进入baidu是秒进,进入谷歌也可以进入..说明已经配置成功...  
好了,可以使用google了,美滋滋,话说...在安装配置ss的过程中,在baidu搜有关ss的几乎都被屏蔽了..不得已使用红莲的电脑使用google搜索...


>参考文章: http://blog.csdn.net/u012810317/article/details/52139361  
> https://github.com/breakwa11/shadowsocks-rss/wiki/Python-client-setup-(Mult-language)